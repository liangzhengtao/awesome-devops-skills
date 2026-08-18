# Deployment Strategies / 部署策略

> Blue-green, canary, rolling, and progressive delivery patterns with real implementations for Kubernetes, AWS, and service meshes.

## When to Use / 何时使用

Use this skill when:

- Choosing between blue-green, canary, rolling, or recreate deployments
- Implementing zero-downtime deployments on Kubernetes
- Setting up progressive delivery with traffic splitting
- Configuring automated rollback on error rate spikes
- Designing deployment pipelines that satisfy SLA requirements
- Migrating from "big bang" releases to safer incremental strategies

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Deployment Strategies                          │
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │   Blue-Green │  │   Canary    │  │   Rolling   │              │
│  │             │  │             │  │             │              │
│  │ Blue (v1) ←─┤  │ v1 (90%)   │  │ v1→v1→v1   │              │
│  │ Green (v2)  │  │ v2 (10%)   │  │ v2→v1→v1   │              │
│  │      ↓      │  │      ↓      │  │ v2→v2→v1   │              │
│  │  Instant    │  │  Gradual    │  │ v2→v2→v2   │              │
│  │  Switch     │  │  Shift      │  │  Incremental│              │
│  └─────────────┘  └─────────────┘  └─────────────┘              │
│                                                                   │
│  Downtime:  None    None         None                            │
│  Risk:      Low     Lowest       Medium                          │
│  Cost:      2x      1.1x        1x                               │
│  Rollback:  Instant Instant      Progressive                     │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Kubernetes Rolling Deployment

```yaml
# deployment-rolling.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # Max pods above desired count
      maxUnavailable: 0     # Zero downtime guarantee
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        version: "2.0.0"
    spec:
      terminationGracePeriodSeconds: 60
      containers:
        - name: app
          image: my-app:2.0.0
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]
```

### 2. Blue-Green Deployment with Argo Rollouts

```yaml
# rollout-blue-green.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: my-app-active
      previewService: my-app-preview
      autoPromotionEnabled: false
      prePromotionAnalysis:
        templates:
          - templateName: success-rate
        args:
          - name: service-name
            value: my-app-preview
      scaleDownDelaySeconds: 300   # Keep old version for quick rollback
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:2.0.0
          ports:
            - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-active
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: my-app-preview
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
```

### 3. Canary Deployment with Traffic Splitting

```yaml
# rollout-canary.yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    canary:
      canaryService: my-app-canary
      stableService: my-app-stable
      trafficRouting:
        nginx:
          stableIngress: my-app-ingress
      steps:
        # Step 1: 10% traffic to canary
        - setWeight: 10
        - pause: { duration: 5m }
        # Step 2: Run analysis
        - analysis:
            templates:
              - templateName: success-rate
            args:
              - name: service-name
                value: my-app-canary
        # Step 3: Increase to 30%
        - setWeight: 30
        - pause: { duration: 5m }
        # Step 4: Increase to 60%
        - setWeight: 60
        - pause: { duration: 5m }
        # Step 5: Full rollout
        - setWeight: 100
      maxSurge: "25%"
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:2.0.0
          ports:
            - containerPort: 8080
```

### 4. Analysis Template for Automated Rollback

```yaml
# analysis-template.yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
    - name: service-name
  metrics:
    - name: success-rate
      interval: 60s
      count: 5
      successCondition: result[0] >= 0.95
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            sum(
              rate(http_requests_total{
                service="{{args.service-name}}",
                status=~"2.."
              }[2m])
            )
            /
            sum(
              rate(http_requests_total{
                service="{{args.service-name}}"
              }[2m])
            )
    - name: latency-p99
      interval: 60s
      count: 5
      successCondition: result[0] <= 500
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            histogram_quantile(0.99,
              sum(rate(http_request_duration_seconds_bucket{
                service="{{args.service-name}}"
              }[2m])) by (le)
            ) * 1000
    - name: error-rate
      interval: 30s
      count: 10
      successCondition: result[0] <= 0.01
      failureLimit: 3
      provider:
        prometheus:
          address: http://prometheus.monitoring:9090
          query: |
            sum(rate(http_requests_total{
              service="{{args.service-name}}",
              status=~"5.."
            }[5m]))
            /
            sum(rate(http_requests_total{
              service="{{args.service-name}}"
            }[5m]))
```

### 5. AWS CodeDeploy Blue-Green

```yaml
# appspec.yml
version: 0.0
os: linux
files:
  - source: /
    destination: /opt/app
hooks:
  BeforeInstall:
    - location: scripts/before_install.sh
      timeout: 300
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
  ValidateService:
    - location: scripts/validate.sh
      timeout: 120
```

```hcl
# deploy.tf - AWS CodeDeploy Blue/Green
resource "aws_codedeploy_app" "app" {
  name             = "my-app"
  compute_platform = "ECS"
}

resource "aws_codedeploy_deployment_group" "production" {
  app_name               = aws_codedeploy_app.app.name
  deployment_group_name  = "production"
  service_role_arn       = aws_iam_role.codedeploy.arn
  deployment_config_name = "CodeDeployDefault.ECSAllAtOnce"

  auto_rollback_configuration {
    enabled = true
    events  = ["DEPLOYMENT_FAILURE", "DEPLOYMENT_STOP_ON_REQUEST"]
  }

  blue_green_deployment_config {
    deployment_ready_option {
      action_on_timeout    = "CONTINUE_DEPLOYMENT"
      wait_time_in_minutes = 0
    }
    terminate_blue_instances_on_deployment_success {
      action                           = "TERMINATE"
      termination_wait_time_in_minutes = 5
    }
  }

  ecs_service {
    cluster_name = aws_ecs_cluster.main.name
    service_name = aws_ecs_service.app.name
  }

  load_balancer_info {
    target_group_pair_info {
      prod_traffic_route {
        listener_arns = [aws_lb_listener.production.arn]
      }
      test_traffic_route {
        listener_arns = [aws_lb_listener.test.arn]
      }
      target_group {
        name = aws_lb_target_group.blue.name
      }
      target_group {
        name = aws_lb_target_group.green.name
      }
    }
  }
}
```

### 6. Progressive Delivery Helmfile

```yaml
# helmfile.yaml
environments:
  staging:
    values:
      - replicas: 2
        maxSurge: 1
  production:
    values:
      - replicas: 5
        maxSurge: 2

releases:
  - name: my-app
    namespace: {{ .Environment.Name }}
    chart: ./charts/my-app
    values:
      - image:
          tag: {{ env "IMAGE_TAG" | default "latest" }}
        replicaCount: {{ .Values.replicas }}
        strategy:
          type: RollingUpdate
          rollingUpdate:
            maxSurge: {{ .Values.maxSurge }}
            maxUnavailable: 0
    hooks:
      - events: ["postsync"]
        command: "scripts/verify-deployment.sh"
        args: ["{{ .Environment.Name }}"]
```

## Proven Patterns / 最佳实践

1. **Always use readiness probes** — Kubernetes routes traffic to pods only after readiness checks pass. Without them, requests hit unready containers.
2. **Implement graceful shutdown** — handle SIGTERM, drain connections, and set `preStop` hooks to allow in-flight requests to complete.
3. **Set `maxUnavailable: 0`** — for zero-downtime rolling deployments, ensure new pods are ready before old ones terminate.
4. **Use analysis templates** — automate canary promotion/rollback based on Prometheus metrics rather than time-based pauses.
5. **Keep blue-green environments identical** — configuration drift between blue and green causes subtle production bugs.
6. **Set appropriate scale-down delays** — keep old versions running for 5-10 minutes after promotion for instant rollback.
7. **Monitor error budgets** — tie deployment velocity to your SLO error budget. If budget is burned, slow down deployments.
8. **Use PodDisruptionBudgets** — ensure voluntary disruptions (node drain, cluster upgrades) don't take down too many replicas.
9. **Implement database migration compatibility** — schema changes must be backward-compatible to support rolling deployments.
10. **Test rollback procedures** — practice rollbacks in staging. A rollback you've never tested is not a rollback strategy.

## Pitfalls / 常见陷阱

1. **Breaking change in API** — deploying a new API version that removes fields breaks running old clients. Always version APIs and make additive changes.
2. **Long-lived connections** — WebSocket and gRPC streams don't respect rolling update termination. Implement connection draining with proper timeout.
3. **Insufficient readiness probe timeout** — if `initialDelaySeconds` is too short, traffic hits pods before the app starts, causing 503 errors.
4. **Database migration locks** — long-running ALTER TABLE locks block both blue and green versions. Use online schema migration tools (gh-ost, pt-online-schema-change).
5. **Canary analysis on low traffic** — statistical significance requires sufficient traffic. A canary at 10% with 100 req/min won't detect P99 latency issues.
6. **Sticky sessions** — if your load balancer uses sticky sessions, canary traffic splitting doesn't work properly. Use stateless session management.
7. **Resource contention during blue-green** — running two full environments simultaneously may exceed cluster capacity. Pre-provision resources.
8. **DNS TTL** — blue-green switches via DNS changes respect TTL caching. Use service mesh or load balancer switching instead.
9. **Health check endpoint lies** — the `/health` endpoint must check downstream dependencies (DB, cache) not just "process is running."
10. **Ignoring deployment velocity** — too-aggressive canary steps (10% → 50% → 100%) skip meaningful observation windows. Match step duration to your metric aggregation interval.

---

## 中文版本

### 使用场景

- 选择蓝绿部署、金丝雀发布、滚动更新或重建部署策略
- 在 Kubernetes 上实现零停机部署
- 配置基于流量分割的渐进式交付
- 设置错误率飙升时的自动回滚
- 设计满足 SLA 要求的部署流水线
- 从"大爆炸"发布迁移到更安全的增量策略

### 核心步骤

1. **滚动更新** — 配置 `maxSurge: 1` + `maxUnavailable: 0` 确保零停机，设置 readiness/liveness probe
2. **蓝绿部署** — 使用 Argo Rollouts 的 `blueGreen` 策略，配置 active/preview service，保留旧版本以便快速回滚
3. **金丝雀发布** — 通过 `canary.steps` 逐步增加流量（10% → 30% → 60% → 100%），配合 Prometheus 指标自动分析
4. **自动回滚分析** — 定义 AnalysisTemplate 监控成功率、P99 延迟、错误率，自动触发回滚
5. **AWS CodeDeploy 蓝绿** — 配置 ECS 蓝绿部署，支持自动回滚和终止旧实例

### 模板说明

- Rolling Deployment — 标准 K8s 滚动更新，包含完整的 probe 和 resource 配置
- Argo Rollouts Blue-Green — 使用 Rollout CRD 实现蓝绿切换
- Argo Rollouts Canary — 多阶段金丝雀发布，配合 traffic routing
- AnalysisTemplate — Prometheus 指标驱动的自动分析和回滚模板

### 常见陷阱

1. **API 破坏性变更** — 新版本移除字段会导致旧客户端报错，API 变更必须向后兼容
2. **长连接不配合滚动更新** — WebSocket/gRPC 流不尊重终止信号，需实现连接排空
3. **Readiness probe 超时过短** — `initialDelaySeconds` 太短会导致流量打到未就绪的 Pod
4. **数据库迁移锁** — 长时间 ALTER TABLE 会阻塞蓝绿两个版本，使用在线迁移工具（gh-ost）
5. **金丝雀流量不足** — 10% 流量下 100 req/min 无法检测 P99 延迟问题，需保证统计显著性
