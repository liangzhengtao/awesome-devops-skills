# GitLab CI/CD / GitLab CI/CD 流水线

> Production-grade GitLab CI/CD pipelines: runners, DAG pipelines, environments, protected variables, and multi-project pipelines.

## When to Use / 何时使用

Use this skill when:

- Building CI/CD pipelines on GitLab (self-hosted or SaaS)
- Migrating from Jenkins or other CI systems to GitLab
- Implementing complex multi-stage pipelines with DAG dependencies
- Setting up GitLab Runner autoscaling on Kubernetes or cloud VMs
- Configuring GitLab environments with review apps
- Integrating container scanning, SAST, DAST into the pipeline

Do NOT use this skill for GitHub Actions — use `github-actions-advanced.md` instead.

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────┐
│                     GitLab Project                            │
│                                                               │
│  .gitlab-ci.yml (main pipeline)                               │
│  ├── stages: [build, test, scan, deploy-staging, deploy-prod] │
│  │                                                            │
│  includes:                                                    │
│  ├── .gitlab/ci/templates/build.yml                           │
│  ├── .gitlab/ci/templates/test.yml                            │
│  ├── .gitlab/ci/templates/security.yml                        │
│  └── .gitlab/ci/templates/deploy.yml                          │
│                                                               │
│  Runners:                                                     │
│  ├── Shared runners (Docker executor)                         │
│  ├── Group runners (autoscaling on AWS/GCP)                   │
│  └── Project runners (GPU, private network)                   │
│                                                               │
│  Pipeline Flow:                                               │
│  push ─► build ─► test ─► scan ─► deploy-staging              │
│                                  └─► (manual) deploy-prod     │
└──────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Full Production Pipeline

```yaml
# .gitlab-ci.yml
variables:
  DOCKER_TLS_CERTDIR: "/certs"
  DOCKER_HOST: tcp://docker:2376
  DOCKER_DRIVER: overlay2

stages:
  - build
  - test
  - scan
  - deploy-staging
  - deploy-production

default:
  image: docker:24-dind
  tags: [docker]
  before_script:
    - echo "$CI_REGISTRY_PASSWORD" | docker login -u "$CI_REGISTRY_USER" --password-stdin "$CI_REGISTRY"

# ── Build ──────────────────────────────────────────────
build:
  stage: build
  script:
    - docker build
        --cache-from "$CI_REGISTRY_IMAGE:latest"
        --tag "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
        --tag "$CI_REGISTRY_IMAGE:latest"
        .
    - docker push "$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA"
    - docker push "$CI_REGISTRY_IMAGE:latest"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

# ── Test ───────────────────────────────────────────────
test:unit:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - npm ci --prefer-offline
    - npm run test -- --coverage --ci
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      junit: reports/junit.xml
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

test:integration:
  stage: test
  image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  services:
    - name: postgres:16-alpine
      alias: db
      variables:
        POSTGRES_DB: testdb
        POSTGRES_USER: testuser
        POSTGRES_PASSWORD: testpass
    - name: redis:7-alpine
      alias: cache
  variables:
    DATABASE_URL: "postgresql://testuser:testpass@db:5432/testdb"
    REDIS_URL: "redis://cache:6379"
  script:
    - npm run test:integration
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

# ── Security Scanning ──────────────────────────────────
include:
  - template: Security/SAST.gitlab-ci.yml
  - template: Security/Dependency-Scanning.gitlab-ci.yml
  - template: Security/Container-Scanning.gitlab-ci.yml
  - template: Security/Secret-Detection.gitlab-ci.yml

# ── Deploy Staging ─────────────────────────────────────
deploy:staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    url: https://staging.example.com
    on_stop: stop:staging
  script:
    - kubectl config use-context staging
    - |
      helm upgrade --install my-app ./charts/my-app \
        --namespace staging \
        --set image.tag=$CI_COMMIT_SHA \
        --set replicas=2 \
        --wait --timeout 300s
    - kubectl rollout status deployment/my-app -n staging --timeout=120s
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

stop:staging:
  stage: deploy-staging
  image: bitnami/kubectl:latest
  environment:
    name: staging
    action: stop
  script:
    - helm uninstall my-app -n staging || true
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual

# ── Deploy Production ──────────────────────────────────
deploy:production:
  stage: deploy-production
  image: bitnami/kubectl:latest
  environment:
    name: production
    url: https://app.example.com
  script:
    - kubectl config use-context production
    - |
      helm upgrade --install my-app ./charts/my-app \
        --namespace production \
        --set image.tag=$CI_COMMIT_SHA \
        --set replicas=5 \
        --set resources.limits.memory=512Mi \
        --wait --timeout 600s
    - kubectl rollout status deployment/my-app -n production --timeout=180s
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
  allow_failure: false
```

### 2. DAG Pipeline (needs/dependencies)

```yaml
# Optimize pipeline duration with explicit dependencies
stages: [build, test, deploy]

build:frontend:
  stage: build
  script: npm run build:frontend
  artifacts:
    paths: [dist/frontend]

build:backend:
  stage: build
  script: npm run build:backend
  artifacts:
    paths: [dist/backend]

test:frontend:
  stage: test
  needs: [build:frontend]
  script: npm run test:frontend

test:backend:
  stage: test
  needs: [build:backend]
  script: npm run test:backend

test:e2e:
  stage: test
  needs: [build:frontend, build:backend]
  script: npm run test:e2e

deploy:
  stage: deploy
  needs: [test:frontend, test:backend, test:e2e]
  script: ./deploy.sh
```

### 3. Runner Configuration (autoscaling)

```toml
# /etc/gitlab-runner/config.toml
concurrent = 10
check_interval = 3

[session_server]
  session_timeout = 1800

[[runners]]
  name = "autoscaling-runner"
  url = "https://gitlab.example.com/"
  token = "RUNNER_TOKEN"
  executor = "docker+machine"
  limit = 20
  
  [runners.docker]
    image = "docker:24-dind"
    privileged = true
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    volumes = ["/certs/client", "/cache"]

  [runners.machine]
    IdleCount = 2
    IdleTime = 600
    MaxBuilds = 100
    MachineName = "runner-%s"
    MachineDriver = "amazonec2"
    MachineOptions = [
      "amazonec2-instance-type=t3.medium",
      "amazonec2-region=us-east-1",
      "amazonec2-ami=ami-0abcdef1234567890",
      "amazonec2-vpc-id=vpc-xxxxxx",
      "amazonec2-subnet-id=subnet-xxxxxx",
      "amazonec2-use-private-address=true",
    ]

    [[runners.machine.autoscaling]]
      Periods = ["* * 8-17 * * mon-fri *"]
      IdleCount = 5
      IdleTime = 300
      Timezone = "America/New_York"
```

### 4. Review Apps with Dynamic Environments

```yaml
review:
  stage: deploy
  image: bitnami/kubectl:latest
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    url: https://$CI_COMMIT_REF_SLUG.review.example.com
    on_stop: review:stop
    auto_stop_in: 1 week
  script:
    - |
      helm upgrade --install review-$CI_COMMIT_REF_SLUG ./charts/my-app \
        --namespace review \
        --set ingress.host=$CI_COMMIT_REF_SLUG.review.example.com \
        --set image.tag=$CI_COMMIT_SHA \
        --set replicas=1
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"

review:stop:
  stage: deploy
  environment:
    name: review/$CI_COMMIT_REF_SLUG
    action: stop
  script:
    - helm uninstall review-$CI_COMMIT_REF_SLUG -n review || true
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: manual
```

### 5. Include Templates and Extends

```yaml
# .gitlab/ci/templates/docker-build.yml
.docker_build:
  image: docker:24-dind
  services:
    - docker:24-dind
  variables:
    DOCKER_BUILDKIT: "1"
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
```

```yaml
# .gitlab-ci.yml
include:
  - local: .gitlab/ci/templates/docker-build.yml

build:api:
  extends: .docker_build
  variables:
    IMAGE_TAG: $CI_REGISTRY_IMAGE/api:$CI_COMMIT_SHA
  before_script:
    - cd services/api
```

## Best Practices / 最佳实践

1. **Use `rules` instead of `only/except`** — `rules` is the modern syntax with better expressiveness and is the recommended approach.
2. **Enable DAG with `needs`** — explicitly declare job dependencies so independent jobs run in parallel instead of sequentially.
3. **Use `include` for modularity** — split large `.gitlab-ci.yml` files into template fragments under `.gitlab/ci/`.
4. **Set `resource_group` for production** — prevents concurrent deployments to the same environment.
5. **Use GitLab's built-in security templates** — `include: template: Security/SAST.gitlab-ci.yml` for zero-config scanning.
6. **Cache with proper keys** — use `$CI_COMMIT_REF_SLUG` for branch-specific caches and `$CI_DEFAULT_BRANCH` for shared fallback.
7. **Use `artifacts:reports`** — coverage reports, JUnit results, and security findings integrate directly into GitLab MR UI.
8. **Protect deployment variables** — mark production secrets as "Protected" and "Masked" in Settings > CI/CD > Variables.
9. **Set pipeline timeouts** — add `timeout:` to prevent stuck jobs from consuming runner capacity.
10. **Use `needs: [job:artifacts]`** — pass artifacts between stages efficiently using DAG-aware artifact downloading.

## Pitfalls / 常见陷阱

1. **`only/except` deprecation** — these keywords are still functional but deprecated. Migrate to `rules` to avoid future breakage.
2. **Docker-in-Docker security** — `privileged: true` in DinD runners is a security risk. Use Kaniko or Buildah for rootless builds when possible.
3. **Cache key collisions** — different branches writing to the same cache key corrupt the cache. Always include `$CI_COMMIT_REF_SLUG` in keys.
4. **Missing `needs` on first DAG job** — if a DAG job doesn't explicitly declare `needs`, it defaults to waiting for all previous stage jobs.
5. **Protected variable visibility** — variables marked "Protected" are only available on protected branches/tags. Deploy jobs on feature branches will fail silently.
6. **Runner token rotation** — GitLab 16+ requires runner authentication tokens. Legacy runner registration tokens are deprecated.
7. **Artifact expiration** — default artifact retention is 30 days. Set explicit `expire_in` for build artifacts to manage storage.
8. **Pipeline quotas on SaaS** — GitLab.com has compute minute limits. Use `interruptible: true` on all jobs to auto-cancel on new pushes.
9. **Helm `--wait` timeout** — on large deployments, the default 300s timeout may not be enough. Monitor rollout status separately.
10. **Environment URL variable interpolation** — `$CI_COMMIT_REF_SLUG` in environment URLs must be lowercase and 63 chars max. Long branch names will break the URL.

---

## 中文版本

### 使用场景

- 在 GitLab（自托管或 SaaS）上搭建 CI/CD 流水线
- 从 Jenkins 或其他 CI 系统迁移到 GitLab
- 实现带有 DAG 依赖的复杂多阶段流水线
- 在 Kubernetes 或云 VM 上配置 GitLab Runner 自动扩缩容
- 配置 GitLab environment 和 review app
- 集成容器扫描、SAST、DAST 安全扫描

### 核心步骤

1. **编写 `.gitlab-ci.yml`** — 定义 stages（build、test、scan、deploy），配置 Docker-in-Docker 构建
2. **配置 DAG 依赖** — 使用 `needs` 关键字声明 job 依赖，实现并行执行
3. **设置 Runner** — 配置 autoscaling runner（docker+machine executor），支持定时扩缩容
4. **集成安全扫描** — 通过 `include` 引入 GitLab 内置 SAST、依赖扫描、容器扫描模板
5. **配置 Review App** — 为 MR 自动创建临时环境，设置自动停止时间

### 模板说明

- 完整生产流水线 — 包含 build、test（unit + integration）、安全扫描、staging/production 部署
- DAG 流水线 — 使用 `needs` 实现 frontend/backend 并行构建和测试
- Runner 配置 — AWS autoscaling runner 的 `config.toml` 完整配置
- Review App — MR 触发的动态环境，支持自动清理

### 常见陷阱

1. **`only/except` 已弃用** — 仍可使用但已标记弃用，应迁移到 `rules` 语法
2. **DinD 安全风险** — `privileged: true` 有安全隐患，生产环境建议使用 Kaniko 或 Buildah
3. **缓存 key 冲突** — 不同分支写入相同缓存 key 会损坏缓存，务必包含 `$CI_COMMIT_REF_SLUG`
4. **Protected variable 可见性** — 标记为 Protected 的变量仅在受保护分支/tag 上可用
5. **Artifact 过期** — 默认保留 30 天，需设置 `expire_in` 管理存储成本
