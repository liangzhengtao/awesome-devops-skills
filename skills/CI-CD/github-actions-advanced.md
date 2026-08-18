# GitHub Actions Advanced / GitHub Actions 高级用法

> Production-grade CI/CD patterns for GitHub Actions: matrix builds, caching, OIDC, reusable workflows, and self-hosted runners.

## When to Use / 何时使用

Use this skill when:

- Setting up CI/CD pipelines on GitHub for the first time
- Optimizing slow GitHub Actions workflows
- Implementing secure deployments with OIDC (no long-lived secrets)
- Building reusable workflow libraries across repositories
- Configuring self-hosted runners for cost or compliance reasons
- Debugging flaky or failing GitHub Actions workflows

Do NOT use this skill for GitLab CI/CD — use `gitlab-cicd.md` instead.

## Architecture / 架构

```
┌─────────────────────────────────────────────────────┐
│                   GitHub Repository                  │
│                                                      │
│  .github/workflows/                                  │
│  ├── ci.yml           # Build + test                 │
│  ├── release.yml      # Semantic release              │
│  ├── deploy.yml       # Environment deployments       │
│  └── reusable/        # Shared workflow fragments     │
│      ├── test.yml                                    │
│      └── build.yml                                   │
│                                                      │
│  Triggers:                                           │
│  push → main ──────► deploy (production)             │
│  PR   ─────────────► ci (lint + test + build)        │
│  tag  ─────────────► release (publish artifacts)     │
└─────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Production CI Pipeline with Matrix Builds

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  pull-requests: write

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  lint:
    name: Lint
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint

  test:
    name: Test (Node ${{ matrix.node }})
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm test -- --coverage
      - uses: actions/upload-artifact@v4
        if: matrix.node == 20
        with:
          name: coverage
          path: coverage/

  build:
    name: Build
    needs: [lint, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/build-push-action@v6
        with:
          context: .
          push: false
          tags: app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### 2. OIDC Deployment (No Long-Lived Secrets)

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

permissions:
  id-token: write   # Required for OIDC
  contents: read

jobs:
  deploy:
    name: Deploy to AWS
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials (OIDC)
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/github-actions-deploy
          aws-region: us-east-1

      - name: Deploy with CloudFormation
        run: |
          aws cloudformation deploy \
            --template-file infra/template.yml \
            --stack-name my-app \
            --capabilities CAPABILITY_IAM \
            --parameter-overrides \
              ImageTag=${{ github.sha }} \
              Environment=production

      - name: Verify deployment
        run: |
          ENDPOINT=$(aws cloudformation describe-stacks \
            --stack-name my-app \
            --query 'Stacks[0].Outputs[?OutputKey==`Endpoint`].OutputValue' \
            --output text)
          curl --retry 5 --retry-delay 10 -sf "$ENDPOINT/health" || exit 1
```

### 3. Reusable Workflow Library

```yaml
# .github/workflows/reusable/build-docker.yml
name: Reusable Docker Build

on:
  workflow_call:
    inputs:
      image_name:
        required: true
        type: string
      dockerfile:
        default: Dockerfile
        type: string
      context:
        default: .
        type: string
    outputs:
      image_digest:
        description: Built image digest
        value: ${{ jobs.build.outputs.digest }}

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      digest: ${{ steps.build.outputs.digest }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: build
        uses: docker/build-push-action@v6
        with:
          context: ${{ inputs.context }}
          file: ${{ inputs.dockerfile }}
          push: true
          tags: ghcr.io/${{ github.repository }}/${{ inputs.image_name }}:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

Usage in another workflow:

```yaml
# .github/workflows/release.yml
jobs:
  build-api:
    uses: ./.github/workflows/reusable/build-docker.yml
    with:
      image_name: api-server
      dockerfile: services/api/Dockerfile
      context: services/api/
    permissions:
      contents: read
      packages: write
```

### 4. Self-Hosted Runner with Labels

```yaml
# For GPU workloads or private network access
jobs:
  gpu-test:
    runs-on: [self-hosted, linux, gpu]
    container:
      image: nvidia/cuda:12.4-runtime-ubuntu22.04
    steps:
      - uses: actions/checkout@v4
      - run: nvidia-smi
      - run: python -m pytest tests/gpu/
```

### 5. Advanced Caching Strategy

```yaml
    steps:
      - uses: actions/checkout@v4

      # Multi-layer dependency cache
      - name: Cache node_modules
        uses: actions/cache@v4
        id: cache-nm
        with:
          path: node_modules
          key: nm-${{ runner.os }}-${{ hashFiles('package-lock.json') }}
          restore-keys: nm-${{ runner.os }}-

      - name: Cache build output
        uses: actions/cache@v4
        with:
          path: .next/cache
          key: build-${{ runner.os }}-${{ hashFiles('**/*.ts', '**/*.tsx') }}
          restore-keys: build-${{ runner.os }}-

      - name: Install dependencies
        if: steps.cache-nm.outputs.cache-hit != 'true'
        run: npm ci

      # Docker layer cache via registry
      - uses: docker/build-push-action@v6
        with:
          cache-from: type=registry,ref=ghcr.io/org/app:buildcache
          cache-to: type=registry,ref=ghcr.io/org/app:buildcache,mode=max
```

### 6. Semantic Release Automation

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

permissions:
  contents: write
  pull-requests: write
  issues: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release
```

## Proven Patterns / 最佳实践

1. **Always set `permissions` explicitly** — least-privilege prevents token abuse. Default is read-write; narrow it down.
2. **Use `concurrency` groups** — cancel redundant runs on the same branch to save runner minutes.
3. **Pin action versions to SHA** — `uses: actions/checkout@abc123` instead of `@v4` for supply-chain safety.
4. **Cache aggressively** — npm, pip, Docker layers, build artifacts. Measure cache hit rates.
5. **Use OIDC instead of long-lived secrets** — `aws-actions/configure-aws-credentials` with role assumption eliminates static keys.
6. **Separate CI from CD** — `ci.yml` runs on every PR; `deploy.yml` only on main branch push.
7. **Use reusable workflows** — DRY principle for Docker builds, deploy steps shared across services.
8. **Set `fail-fast: false` in matrix** — get results from all matrix legs, not just the first failure.
9. **Use GitHub Environments** — add approval gates, deployment protection rules, and environment-scoped secrets.
10. **Monitor costs** — use `gh api /repos/{owner}/{repo}/actions/workflows/{id}/timing` to track workflow run times.

## Pitfalls / 常见陷阱

1. **Missing `fetch-depth: 0`** — Semantic release and changelog generation need full git history. Shallow clones break them.
2. **Secrets in forks** — PRs from forks don't have access to repository secrets by default. Design CI to work without secrets for fork PRs.
3. **Stale caches** — Key caches with `hashFiles()` that cover the actual dependency surface. A too-broad key causes cache pollution.
4. **Runner label mismatches** — Self-hosted runners need exact label matches. A typo in `runs-on: [self-hosted, linux, gpu]` silently queues forever.
5. **Environment protection without reviewers** — An environment with required reviewers but no assigned reviewers blocks deployments indefinitely.
6. **Docker layer cache invalidation** — Copy `package.json` and install deps before copying source code in Dockerfiles to maximize layer reuse.
7. **Rate limiting on GitHub API** — Workflows calling `gh api` in loops hit secondary rate limits. Use `GITHUB_TOKEN` and batch requests.
8. **OIDC thumbprint rotation** — AWS OIDC provider thumbprints expire. Set up monitoring and rotation for the GitHub OIDC provider.
9. **Artifact retention** — Default artifact retention is 90 days. Set `retention-days` to avoid storage costs for build artifacts.
10. **Workflow file size limit** — GitHub limits workflow files to 65535 bytes. Split complex workflows using reusable workflow calls.

---

## 中文版本

### 使用场景

- 首次在 GitHub 上搭建 CI/CD 流水线时
- 优化慢速 GitHub Actions workflow
- 使用 OIDC 实现安全部署（无需长期密钥）
- 跨仓库构建可复用的 workflow 库
- 出于成本或合规原因配置 self-hosted runner
- 调试不稳定的 GitHub Actions workflow

> 注意：GitLab CI/CD 请使用 `gitlab-cicd.md`。

### 核心步骤

1. **配置 CI 流水线** — 使用 matrix build 覆盖多版本 Node.js，设置 `concurrency` 取消冗余运行
2. **启用 OIDC 部署** — 通过 `id-token: write` 权限实现无密钥 AWS 部署，避免长期 secret
3. **构建可复用 workflow** — 将 Docker build、deploy 等步骤封装为 `workflow_call`，跨服务复用
4. **配置缓存策略** — 使用 `actions/cache` 多层缓存 npm 依赖和构建产物，利用 GHA cache 缓存 Docker layer
5. **自动发布** — 配合 `semantic-release` 实现版本自动管理和 changelog 生成

### 模板说明

- `ci.yml` — 包含 lint、test（matrix）、build 三个 job，使用 GHA cache 加速 Docker build
- `deploy.yml` — OIDC 部署模板，支持 CloudFormation 部署和健康检查验证
- `build-docker.yml` — 可复用 Docker 构建 workflow，支持自定义 Dockerfile 和 context
- `release.yml` — semantic-release 自动发布模板

### 常见陷阱

1. **`fetch-depth` 不足** — semantic release 需要完整 git history，浅克隆会导致失败
2. **Fork PR 无 secret** — 来自 fork 的 PR 默认无法访问仓库 secret，CI 设计需兼容无 secret 场景
3. **缓存 key 过宽** — `hashFiles()` 需精确覆盖依赖面，key 过宽会导致缓存污染
4. **Runner label 拼写错误** — self-hosted runner 需精确匹配 label，拼写错误会导致 job 静默排队
5. **Environment 无审批人** — 配置了 required reviewer 但未分配审批人，部署会无限阻塞
