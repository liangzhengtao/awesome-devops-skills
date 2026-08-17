# Docker for Production / 生产环境 Docker 最佳实践

> Multi-stage builds, security hardening, image optimization, health checks, logging, and resource management for production Docker containers.

## When to Use / 何时使用

Use this skill when:

- Writing or optimizing Dockerfiles for production workloads
- Reducing Docker image sizes and build times
- Hardening container security (rootless, read-only, capabilities)
- Implementing proper health checks and graceful shutdown
- Setting up Docker logging and monitoring
- Debugging container startup failures or resource issues
- Building multi-architecture images (amd64 + arm64)

## Architecture / 架构

```
┌────────────────────────────────────────────────────────────────┐
│                   Production Docker Architecture                │
│                                                                 │
│  Build Phase          Runtime Phase                             │
│  ┌───────────┐        ┌──────────────────────────┐             │
│  │ Multi-stage│        │ Container Runtime         │             │
│  │ Build      │        │                           │             │
│  │           │        │  ┌─────────────────────┐  │             │
│  │ Stage 1:  │        │  │ Non-root user        │  │             │
│  │ Build deps│───►    │  │ Read-only rootfs      │  │             │
│  │           │  image │  │ No new privileges     │  │             │
│  │ Stage 2:  │        │  │ Resource limits       │  │             │
│  │ Runtime   │        │  │ Health checks         │  │             │
│  │ only      │        │  │ Graceful shutdown     │  │             │
│  └───────────┘        │  └─────────────────────┘  │             │
│                       └──────────────────────────┘             │
│                                                                 │
│  Image Size:  1.2GB (dev) → 45MB (production)                  │
│  Attack Surface: npm + dev tools → busybox/alpine only         │
└────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Multi-Stage Node.js Build

```dockerfile
# ── Stage 1: Dependencies ──────────────────────────
FROM node:20-alpine AS deps
WORKDIR /app

# Copy only dependency files for better caching
COPY package.json package-lock.json ./
RUN npm ci --prefer-offline --ignore-scripts && \
    npm cache clean --force

# ── Stage 2: Build ─────────────────────────────────
FROM node:20-alpine AS build
WORKDIR /app

COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npm run build && \
    npm prune --omit=dev && \
    rm -rf src/ tests/ .git/ *.md tsconfig*.json

# ── Stage 3: Production ────────────────────────────
FROM node:20-alpine AS production
WORKDIR /app

# Security: non-root user
RUN addgroup -g 1001 -S app && \
    adduser -S app -u 1001 -G app

# Copy only production artifacts
COPY --from=build --chown=app:app /app/dist ./dist
COPY --from=build --chown=app:app /app/node_modules ./node_modules
COPY --from=build --chown=app:app /app/package.json ./

# Runtime configuration
ENV NODE_ENV=production \
    PORT=3000

EXPOSE 3000

USER app

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

CMD ["node", "dist/server.js"]
```

### 2. Python Application Dockerfile

```dockerfile
# ── Build Stage ─────────────────────────────────────
FROM python:3.12-slim AS builder
WORKDIR /app

RUN pip install --no-cache-dir --upgrade pip setuptools wheel

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt

# ── Production Stage ────────────────────────────────
FROM python:3.12-slim AS production

# Install only runtime dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends curl && \
    rm -rf /var/lib/apt/lists/*

# Non-root user
RUN groupadd -r app && useradd -r -g app -d /app -s /sbin/nologin app

WORKDIR /app

# Copy installed packages
COPY --from=builder /install /usr/local
COPY --chown=app:app . .

USER app

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8000

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --start-period=15s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

CMD ["gunicorn", "app.main:app", \
     "--bind", "0.0.0.0:8000", \
     "--workers", "4", \
     "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--timeout", "120"]
```

### 3. Go Application (Scratch/Distroless)

```dockerfile
# ── Build Stage ─────────────────────────────────────
FROM golang:1.22-alpine AS builder
WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -ldflags="-w -s -X main.version=$(cat VERSION)" \
    -o /app/server ./cmd/server

# ── Production Stage (distroless) ───────────────────
FROM gcr.io/distroless/static-debian12:nonroot

COPY --from=builder /app/server /server

EXPOSE 8080

ENTRYPOINT ["/server"]
```

### 4. Docker Compose Production Stack

```yaml
# docker-compose.prod.yml
services:
  app:
    image: ${REGISTRY}/my-app:${TAG:-latest}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: "1.0"
          memory: 512M
        reservations:
          cpus: "0.25"
          memory: 128M
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
    read_only: true
    tmpfs:
      - /tmp:size=100M
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 15s
    environment:
      - NODE_ENV=production
    networks:
      - app-net
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"

  redis:
    image: redis:7-alpine
    deploy:
      resources:
        limits:
          memory: 256M
    read_only: true
    command: redis-server --maxmemory 200mb --maxmemory-policy allkeys-lru
    volumes:
      - redis-data:/data
    networks:
      - app-net

volumes:
  redis-data:

networks:
  app-net:
    driver: bridge
```

### 5. Container Security Scanning

```yaml
# .github/workflows/scan.yml
name: Container Security Scan

on:
  push:
    branches: [main]
  pull_request:

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t my-app:scan .

      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: my-app:scan
          format: table
          exit-code: 1           # Fail on HIGH/CRITICAL
          severity: HIGH,CRITICAL
          ignore-unfixed: true

      - name: Run Dockle lint
        uses: goodwithtech/dockle-action@main
        with:
          image: my-app:scan
          exit-code: 1
          exit-level: warn
```

### 6. Multi-Architecture Build

```dockerfile
# Dockerfile.multiarch
FROM --platform=$BUILDPLATFORM golang:1.22-alpine AS builder
ARG TARGETOS TARGETARCH
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=$TARGETOS GOARCH=$TARGETARCH \
    go build -o /server ./cmd/server

FROM gcr.io/distroless/static-debian12:nonroot
COPY --from=builder /server /server
ENTRYPOINT ["/server"]
```

```bash
# Build and push multi-arch
docker buildx create --use --name multiarch
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t registry/my-app:latest \
  --push .
```

## Best Practices / 最佳实践

1. **Use multi-stage builds** — separate build dependencies from runtime. A Node.js app drops from 1.2GB to 45MB.
2. **Run as non-root** — `USER 1001` prevents container breakout attacks from gaining root on the host.
3. **Use `.dockerignore`** — exclude `.git`, `node_modules`, tests, docs. Reduces build context from 500MB to 5MB.
4. **Pin base image digests** — `FROM node:20-alpine@sha256:abc123` ensures reproducible builds.
5. **Copy dependency files first** — `COPY package.json` before `COPY .` maximizes Docker layer cache hits.
6. **Set `HEALTHCHECK`** — Docker and orchestrators use health checks to route traffic and restart unhealthy containers.
7. **Use `distroless` or `alpine`** — smaller images = faster pulls, smaller attack surface, fewer CVEs.
8. **Enable BuildKit** — `DOCKER_BUILDKIT=1` enables parallel builds, better caching, and secret mounts.
9. **Set resource limits** — always set CPU and memory limits to prevent noisy neighbor problems.
10. **Use `--no-cache` for package installs** — `npm ci --ignore-scripts && npm cache clean --force` keeps the layer small.

## Pitfalls / 常见陷阱

1. **Running as root** — the default Docker user is root. Always add `USER` directive. Many images silently run as root.
2. **COPY . . before dependency install** — copying all source before `npm ci` invalidates the dependency cache on every code change.
3. **No `.dockerignore`** — sending 500MB of `.git` and `node_modules` to the build daemon on every build.
4. **Using `latest` tag in production** — `latest` is mutable. Use semantic version tags or image digests.
5. **Not handling SIGTERM** — if your app doesn't handle SIGTERM, Docker sends SIGKILL after the grace period, corrupting in-flight requests.
6. **Secrets in build args** — `ARG SECRET` is visible in `docker history`. Use `--mount=type=secret` with BuildKit.
7. **Alpine + glibc issues** — Alpine uses musl libc. Some Python/Node native packages fail to compile. Use `-slim` variants if needed.
8. **Layer ordering** — put frequently changing layers (COPY source) after rarely changing layers (COPY package.json) to maximize cache.
9. **Health check too aggressive** — a health check that runs every 1s with a 1s timeout wastes CPU. Use 30s intervals with 5s timeouts.
10. **No graceful shutdown** — containers need time to drain connections. Set `STOPSIGNAL SIGTERM` and handle it in application code.
