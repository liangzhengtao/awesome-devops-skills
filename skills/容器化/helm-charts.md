# Helm Chart Management / Helm Chart 管理

> Helm chart development, templating, values management, testing, repository setup, and production deployment patterns.

## When to Use / 何时使用

Use this skill when:

- Creating Helm charts for new applications
- Managing Kubernetes manifests with templating and value overrides
- Setting up Helm chart repositories (ChartMuseum, OCI, GitHub Pages)
- Testing Helm charts with unit tests and integration tests
- Managing multi-environment deployments (dev, staging, prod)
- Upgrading and rolling back Helm releases
- Structuring Helm charts for microservices

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────┐
│                    Helm Chart Structure                        │
│                                                               │
│  my-app/                                                     │
│  ├── Chart.yaml          # Chart metadata + dependencies     │
│  ├── Chart.lock           # Dependency lock file             │
│  ├── values.yaml          # Default values                  │
│  ├── values-staging.yaml  # Environment overrides            │
│  ├── values-prod.yaml     # Production overrides             │
│  ├── templates/           # K8s manifest templates           │
│  │   ├── _helpers.tpl     # Template helpers                │
│  │   ├── deployment.yaml                                     │
│  │   ├── service.yaml                                        │
│  │   ├── ingress.yaml                                        │
│  │   ├── hpa.yaml                                            │
│  │   ├── pdb.yaml                                            │
│  │   ├── serviceaccount.yaml                                 │
│  │   ├── configmap.yaml                                      │
│  │   ├── secret.yaml                                         │
│  │   ├── NOTES.txt          # Post-install notes             │
│  │   └── tests/                                             │
│  │       └── test-connection.yaml                             │
│  ├── charts/              # Subchart dependencies            │
│  └── ci/                  # CI test values                   │
│      └── test-values.yaml                                    │
└──────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Chart.yaml with Dependencies

```yaml
# Chart.yaml
apiVersion: v2
name: my-app
description: Production-ready Helm chart for My Application
type: application
version: 1.2.0        # Chart version
appVersion: "2.5.0"   # Application version

maintainers:
  - name: Platform Team
    email: platform@example.com

keywords: [api, microservice, production]

home: https://github.com/org/my-app
sources:
  - https://github.com/org/my-app

dependencies:
  - name: postgresql
    version: "15.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: postgresql.enabled
  - name: redis
    version: "19.x.x"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
  - name: common
    version: "2.x.x"
    repository: https://charts.bitnami.com/bitnami
```

### 2. values.yaml (Comprehensive)

```yaml
# values.yaml
replicaCount: 3

image:
  repository: registry.example.com/my-app
  tag: ""  # Defaults to Chart.appVersion
  pullPolicy: IfNotPresent

imagePullSecrets:
  - name: registry-credentials

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  name: ""
  annotations: {}

podAnnotations:
  prometheus.io/scrape: "true"
  prometheus.io/port: "8080"
  prometheus.io/path: "/metrics"

podSecurityContext:
  runAsNonRoot: true
  runAsUser: 1001
  runAsGroup: 1001
  fsGroup: 1001
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop: [ALL]

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: false
  className: nginx
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: false
  minReplicas: 3
  maxReplicas: 20
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

pdb:
  enabled: true
  minAvailable: 2

nodeSelector: {}
tolerations: []
affinity: {}

topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule

env: {}
envFrom: []

livenessProbe:
  httpGet:
    path: /health/live
    port: http
  initialDelaySeconds: 15
  periodSeconds: 20

readinessProbe:
  httpGet:
    path: /health/ready
    port: http
  initialDelaySeconds: 5
  periodSeconds: 10

postgresql:
  enabled: false
  auth:
    database: myapp
    username: myapp

redis:
  enabled: false
  architecture: standalone
```

### 3. Deployment Template

```yaml
# templates/deployment.yaml
{{- include "my-app.validate" . -}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
spec:
  {{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
  {{- end }}
  revisionHistoryLimit: 5
  selector:
    matchLabels:
      {{- include "my-app.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      annotations:
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "my-app.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      terminationGracePeriodSeconds: 60
      {{- with .Values.topologySpreadConstraints }}
      topologySpreadConstraints:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          {{- with .Values.livenessProbe }}
          livenessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          {{- with .Values.readinessProbe }}
          readinessProbe:
            {{- toYaml . | nindent 12 }}
          {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          envFrom:
            {{- with .Values.envFrom }}
            {{- toYaml . | nindent 12 }}
            {{- end }}
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            {{- range $key, $value := .Values.env }}
            - name: {{ $key }}
              value: {{ $value | quote }}
            {{- end }}
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir:
            sizeLimit: 100Mi
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

### 4. Template Helpers

```yaml
# templates/_helpers.tpl
{{/*
Expand the name of the chart.
*/}}
{{- define "my-app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "my-app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "my-app.labels" -}}
helm.sh/chart: {{ include "my-app.chart" . }}
{{ include "my-app.selectorLabels" . }}
app.kubernetes.io/version: {{ .Values.image.tag | default .Chart.AppVersion | quote }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Chart label
*/}}
{{- define "my-app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Service account name
*/}}
{{- define "my-app.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "my-app.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}

{{/*
Validation
*/}}
{{- define "my-app.validate" -}}
{{- if and .Values.autoscaling.enabled (not .Values.resources.requests.cpu) }}
{{- fail "autoscaling requires cpu requests to be set" }}
{{- end }}
{{- end }}
```

### 5. Multi-Environment Deployment

```bash
# Deploy to staging
helm upgrade --install my-app ./charts/my-app \
  --namespace staging \
  --values charts/my-app/values.yaml \
  --values charts/my-app/values-staging.yaml \
  --set image.tag=$TAG \
  --wait --timeout 300s

# Deploy to production with history
helm upgrade --install my-app ./charts/my-app \
  --namespace production \
  --values charts/my-app/values.yaml \
  --values charts/my-app/values-prod.yaml \
  --set image.tag=$TAG \
  --atomic \
  --wait --timeout 600s \
  --history-max 10

# Rollback to previous version
helm rollback my-app 0 --namespace production

# View release history
helm history my-app --namespace production

# Diff before applying
helm diff upgrade my-app ./charts/my-app \
  --namespace production \
  --values charts/my-app/values-prod.yaml \
  --set image.tag=$TAG
```

### 6. Chart Testing

```yaml
# templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-app.fullname" . }}-test-connection"
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  restartPolicy: Never
  containers:
    - name: wget
      image: busybox:1.36
      command: ['wget']
      args:
        - '--no-verbose'
        - '--tries=3'
        - '--timeout=10'
        - '-O'
        - '/dev/null'
        - '{{ include "my-app.fullname" . }}:{{ .Values.service.port }}/health'
```

```bash
# Run chart tests
helm test my-app --namespace production --timeout 120s

# Lint chart
helm lint ./charts/my-app --values charts/my-app/values-prod.yaml

# Template rendering (dry-run)
helm template my-app ./charts/my-app \
  --namespace production \
  --values charts/my-app/values-prod.yaml \
  --debug
```

## Best Practices / 最佳实践

1. **Use `--atomic` for production** — automatically rolls back on failure. Never deploy without it in production.
2. **Set `checksum/config` annotation** — hash ConfigMaps into pod annotations to trigger rolling restarts on config change.
3. **Use `helm-diff` plugin** — preview changes before applying. Catches unintended deletions and modifications.
4. **Store charts in OCI registries** — `helm push my-app-1.0.0.tgz oci://registry.example.com/charts` is the modern approach.
5. **Version charts semantically** — bump `version` (chart changes) and `appVersion` (app changes) independently.
6. **Use `values-<env>.yaml` overrides** — keep defaults in `values.yaml`, override per environment. Never hardcode env-specific values.
7. **Template with `_helpers.tpl`** — DRY principle for labels, names, and selectors. Every chart should have helpers.
8. **Add chart tests** — `helm test` runs test pods to verify the deployment works. Add connectivity and health tests.
9. **Limit release history** — `--history-max 10` prevents ConfigMap/Secret accumulation from old revisions.
10. **Use `helm secrets` for sensitive values** — encrypt secrets with SOPS before storing in git.

## Pitfalls / 常见陷阱

1. **`helm upgrade` without `--install`** — if the release doesn't exist, `upgrade` fails. Always use `upgrade --install`.
2. **Missing `--wait` flag** — without `--wait`, Helm returns success before pods are ready. Always use `--wait` with a timeout.
3. **Value type mismatches** — `--set replicas=3` creates a string. Use `--set replicas=3` (Helm 3 auto-detects) or `--set-json`.
4. **Subchart version pinning** — without `Chart.lock`, `helm dependency update` may pull different subchart versions. Commit the lock file.
5. **Template rendering order** — Helm renders templates in alphabetical order, not by resource kind. Use hooks for ordering.
6. **Large values files** — monolithic values.yaml with 500+ lines is hard to maintain. Split into logical sections with comments.
7. **`tpl` function performance** — excessive `tpl` calls slow rendering. Use direct value references when possible.
8. **Secret encryption at rest** — K8s Secrets are base64-encoded, not encrypted. Enable etcd encryption or use external secret management.
9. **Chart size limit** — Helm releases have a 1MB limit for the release object. Large ConfigMaps or many resources can exceed this.
10. **CRD management** — Helm 3 doesn't upgrade CRDs automatically. Use a separate chart or `kubectl apply` for CRD updates.

---

## 中文版本

### 使用场景

- 为新应用创建 Helm chart
- 使用模板和 values override 管理 Kubernetes 清单
- 搭建 Helm chart 仓库（ChartMuseum、OCI、GitHub Pages）
- 使用单元测试和集成测试测试 Helm chart
- 管理多环境部署（dev、staging、prod）

### 核心步骤

1. **创建 Chart 结构** — `Chart.yaml` 定义元数据和依赖，`values.yaml` 定义默认值
2. **编写 Deployment 模板** — 使用 `_helpers.tpl` 定义标签、名称等辅助函数，实现 DRY
3. **多环境部署** — `values-staging.yaml` / `values-prod.yaml` 覆盖默认值，`--set` 动态注入 image tag
4. **配置 checksum** — 将 ConfigMap hash 注入 Pod annotation，配置变更时自动触发滚动重启
5. **Chart 测试** — 编写 `test-connection.yaml` 验证部署后的连通性，使用 `helm test` 执行

### 模板说明

- Chart.yaml — 包含 PostgreSQL 和 Redis 依赖的完整 chart 元数据
- values.yaml — 涵盖所有常见配置项的生产级默认值
- Deployment 模板 — 带校验、安全上下文、探针的完整 Deployment
- _helpers.tpl — 标签、名称、ServiceAccount 等辅助模板

### 常见陷阱

1. **`helm upgrade` 不带 `--install`** — release 不存在时 `upgrade` 会失败，始终使用 `upgrade --install`
2. **缺少 `--wait` 标志** — 不用 `--wait` 时 Helm 在 Pod 就绪前就返回成功
3. **Value 类型不匹配** — `--set replicas=3` 可能创建字符串，注意类型推断
4. **子 chart 版本未锁定** — 没有 `Chart.lock` 时 `helm dependency update` 可能拉取不同版本
5. **CRD 不自动升级** — Helm 3 不会自动升级 CRD，需单独用 `kubectl apply` 管理
