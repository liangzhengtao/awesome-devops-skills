# ELK/Loki Logging Stack / ELK/Loki 日志系统

> Centralized logging with Elasticsearch+Kibana (ELK) and Grafana Loki, including log parsing, querying, retention, and cost optimization.

## When to Use / 何时使用

Use this skill when:

- Setting up centralized logging for microservices
- Choosing between ELK stack and Grafana Loki
- Configuring log shipping with Fluentd, Fluent Bit, or Vector
- Writing LogQL (Loki) or KQL/Lucene (Elasticsearch) queries
- Parsing structured and unstructured log formats
- Setting up log-based alerts and dashboards
- Managing log retention and storage costs
- Debugging application issues through log analysis

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                   Logging Architecture (Loki)                     │
│                                                                   │
│  ┌─────────────┐     ┌──────────┐     ┌─────────────┐           │
│  │ Application │────►│ Fluent   │────►│ Grafana     │           │
│  │ (stdout)    │     │ Bit /    │     │ Loki        │           │
│  └─────────────┘     │ Vector   │     │             │           │
│                      └──────────┘     │ ┌─────────┐ │           │
│  ┌─────────────┐                      │ │ Ingester│ │           │
│  │ Nginx Logs  │──────────────────────►│ │         │ │           │
│  └─────────────┘                      │ │ Querier │ │           │
│                                       │ │         │ │           │
│  ┌─────────────┐     ┌──────────┐     │ │Compactor│ │           │
│  │ System Logs │────►│ Promtail │────►│ └─────────┘ │           │
│  └─────────────┘     └──────────┘     └──────┬──────┘           │
│                                              │                    │
│                                         ┌────┴────┐              │
│                                         │ Grafana │              │
│                                         └─────────┘              │
│                                              │                    │
│                                     Object Storage (S3)          │
│                                                                   │
│  ── OR: ELK Stack ──                                             │
│  Fluentd ──► Elasticsearch ──► Kibana                            │
│             (Hot/Warm/Cold)    (Dashboards, Discover)            │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Grafana Loki (Helm Deployment)

```yaml
# loki-values.yaml
loki:
  auth_enabled: false
  commonConfig:
    replication_factor: 1
  storage:
    type: s3
    s3:
      endpoint: s3.us-east-1.amazonaws.com
      region: us-east-1
      bucketnames: loki-chunks
  schemaConfig:
    configs:
      - from: "2024-01-01"
        store: tsdb
        object_store: s3
        schema: v13
        index:
          prefix: loki_index_
          period: 24h

  limits_config:
    retention_period: 30d
    max_query_length: 721h
    max_query_parallelism: 32
    ingestion_rate_mb: 10
    ingestion_burst_size_mb: 20
    per_stream_rate_limit: 5MB
    per_stream_rate_limit_burst: 15MB

  compactor:
    retention_enabled: true
    delete_request_store: s3

  rulerConfig:
    alertmanager_url: http://prometheus-alertmanager.monitoring:9093

read:
  replicas: 3
  resources:
    requests:
      cpu: 500m
      memory: 1Gi

write:
  replicas: 3
  resources:
    requests:
      cpu: 500m
      memory: 1Gi

gateway:
  replicas: 2
```

### 2. Fluent Bit Configuration (DaemonSet)

```yaml
# fluent-bit-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: logging
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         5
        Log_Level     warning
        Daemon        off
        Parsers_File  parsers.conf
        HTTP_Server   On
        HTTP_Listen   0.0.0.0
        HTTP_Port     2020

    # ── Kubernetes logs ─────────────────────────────
    [INPUT]
        Name              tail
        Tag               kube.*
        Path              /var/log/containers/*.log
        Parser            cri
        DB                /var/log/flb_kube.db
        Mem_Buf_Limit     50MB
        Skip_Long_Lines   On
        Refresh_Interval  10
        Rotate_Wait       30

    # ── Kubernetes metadata ─────────────────────────
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Kube_Tag_Prefix     kube.var.log.containers.
        Merge_Log           On
        Merge_Log_Key       log_processed
        K8S-Logging.Parser  On
        K8S-Logging.Exclude On

    # ── Namespace filtering ─────────────────────────
    [FILTER]
        Name    grep
        Match   kube.*
        Exclude $kubernetes['namespace_name'] ^(kube-system|logging)$

    # ── Output to Loki ──────────────────────────────
    [OUTPUT]
        Name                   loki
        Match                  kube.*
        Host                   loki-gateway.logging.svc.cluster.local
        Port                   80
        Labels                 job=fluent-bit
        Label_Keys             $kubernetes['namespace_name'],$kubernetes['pod_name'],$kubernetes['container_name'],$stream
        Remove_Keys            $kubernetes['pod_id'],$kubernetes['docker_id']
        Auto_Kubernetes_Labels Off
        Tenant_ID              ""
        Batch_Wait             1s
        Batch_Size             1048576
        Line_Format            key_value

  parsers.conf: |
    [PARSER]
        Name        cri
        Format      regex
        Regex       ^(?<time>[^ ]+) (?<stream>stdout|stderr) (?<logtag>[^ ]*) (?<log>.*)$
        Time_Key    time
        Time_Format %Y-%m-%dT%H:%M:%S.%L%z

    [PARSER]
        Name        json
        Format      json
        Time_Key    timestamp
        Time_Format %Y-%m-%dT%H:%M:%S.%L%z
```

### 3. Vector (Modern Alternative to Fluentd)

```toml
# vector.toml
# Vector replaces Fluentd/Fluent Bit with Rust performance
data_dir = "/var/lib/vector"

[sources.kubernetes_logs]
type = "kubernetes_logs"
auto_partial_merge = true

# Parse JSON logs
[transforms.parse_json]
type = "remap"
inputs = ["kubernetes_logs"]
source = '''
  . = merge(., parse_json!(.message))
  .level = .level ?? "info"
  .timestamp = .timestamp ?? now()
'''

# Filter noisy logs
[transforms.filter]
type = "filter"
inputs = ["parse_json"]
condition = '''
  !includes(["healthz", "readyz"], .http_request?.path ?? "")
'''

# Enrich with metadata
[transforms.enrich]
type = "remap"
inputs = ["filter"]
source = '''
  .service = .kubernetes?.pod_labels?."app.kubernetes.io/name" ?? "unknown"
  .environment = .kubernetes?.pod_labels?."app.kubernetes.io/instance" ?? "unknown"
'''

# Send to Loki
[sinks.loki]
type = "loki"
inputs = ["enrich"]
endpoint = "http://loki-gateway:80"
encoding.codec = "json"
labels.job = "vector"
labels.namespace = "{{ kubernetes.pod_namespace }}"
labels.service = "{{ service }}"
labels.level = "{{ level }}"

# Health metrics for Vector itself
[sources.vector_metrics]
type = "internal_metrics"

[sinks.prometheus]
type = "prometheus_exporter"
inputs = ["vector_metrics"]
address = "0.0.0.0:9598"
```

### 4. LogQL Query Examples (Loki)

```logql
# ── Basic Queries ───────────────────────────────────
# All logs from a namespace
{namespace="production"}

# Logs from a specific container
{namespace="production", container="api-server"}

# Filter by log level
{namespace="production"} |= "error" | logfmt

# JSON parsing
{namespace="production"} | json | level="error"

# Pattern parsing (Nginx access logs)
{job="nginx"} | pattern `<ip> - - [<ts>] "<method> <path> <_>" <status> <size>`

# ── Advanced Queries ────────────────────────────────
# Error rate per service (5m)
sum(rate({namespace="production"} |= "error" [5m])) by (container)

# Top 10 error-producing pods
topk(10,
  sum(rate({namespace="production"} |= "error" [5m])) by (pod)
)

# Latency distribution from JSON logs
{namespace="production"} | json
  | latency >= 1000
  | line_format "{{.method}} {{.path}} {{.latency}}ms"

# Compare error rates between two time ranges
sum(rate({namespace="production"} |= "error" [1h] offset 1h))
/
sum(rate({namespace="production"} |= "error" [1h]))

# Label filter with regex
{namespace=~"prod.*"} |~ "timeout|deadline exceeded"

# Unwrap for numeric aggregation
{namespace="production"} | json
  | unwrap duration
  | quantile_over_time(0.99, ({namespace="production"} | json | unwrap duration) [5m]) by (service)
```

### 5. Log-Based Alerts (Loki Ruler)

```yaml
# loki-alert-rules.yaml
groups:
  - name: application-alerts
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate({namespace="production"} |= "error" [5m])) by (container)
          /
          sum(rate({namespace="production"} [5m])) by (container)
          > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High error rate in {{ $labels.container }}"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: ErrorSpike
        expr: |
          sum(rate({namespace="production"} |= "error" [5m]))
          > 3 * sum(rate({namespace="production"} |= "error" [5m] offset 1h))
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Error rate 3x higher than same time yesterday"

      - alert: PodOOMKilled
        expr: |
          sum_over_time({namespace="production"} |= "OOMKilled" [5m]) > 0
        labels:
          severity: critical
        annotations:
          summary: "Pod OOMKilled detected"
```

### 6. Elasticsearch Index Lifecycle (ILM)

```json
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_age": "1d",
            "max_primary_shard_size": "50gb"
          },
          "set_priority": { "priority": 100 }
        }
      },
      "warm": {
        "min_age": "3d",
        "actions": {
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 },
          "set_priority": { "priority": 50 }
        }
      },
      "cold": {
        "min_age": "14d",
        "actions": {
          "searchable_snapshot": {
            "snapshot_repository": "found-snapshots"
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

## Proven Patterns / 最佳实践

1. **Write logs to stdout/stderr** — let the container runtime handle log collection. Never write to files inside containers.
2. **Use structured logging (JSON)** — `{"level":"error","msg":"failed","user_id":123}` is parseable and queryable. Free-text is not.
3. **Choose Loki for cost efficiency** — Loki indexes labels only, not log content. 10x cheaper than Elasticsearch for the same volume.
4. **Use ELK for full-text search** — if you need complex queries on log content (regex, fuzzy matching), Elasticsearch is superior.
5. **Set retention policies** — hot: 3 days, warm: 14 days, cold/archive: 30 days, delete: 90 days. Match compliance requirements.
6. **Filter noisy logs at the edge** — health checks, readiness probes, and debug logs should be filtered by Fluent Bit/Vector before shipping.
7. **Use label discipline in Loki** — only index high-cardinality labels you query by (namespace, pod, container). Avoid user_id.
8. **Monitor log pipeline health** — track log ingestion rate, lag, and drops. A silent logging pipeline is a broken one.
9. **Use `line_format` in Loki** — restructure log lines for readability in Grafana. Include service, level, and request ID.
10. **Cost control with limits** — set per-tenant ingestion limits and rate limits to prevent runaway logging from one service.

## Pitfalls / 常见陷阱

1. **Logging everything at DEBUG** — DEBUG logs in production create 10x volume. Set production to INFO/WARN.
2. **No log correlation** — without request IDs (trace_id), it's impossible to follow a request across services. Inject trace IDs.
3. **Elasticsearch cluster overload** — indexing high-volume logs without ILM causes unbounded growth. Disk full = cluster red.
4. **Loki high cardinality labels** — indexing `user_id` or `request_id` as labels causes Loki memory explosion. Use parsed labels for filtering.
5. **Missing timestamp parsing** — if the logging pipeline doesn't parse timestamps, logs show ingestion time, not event time.
6. **Log rotation in containers** — if you write to files inside containers, log rotation tools (logrotate) don't work with ephemeral storage.
7. **Grep-based alerting is fragile** — `|= "error"` matches "error-free", "error-handler", and "error". Use `| json | level="error"` for precision.
8. **No sampling for high-volume logs** — logging every request in a high-traffic API creates TB/day. Use head or tail sampling.
9. **Elasticsearch heap sizing** — ES heap should be 50% of available RAM, max 32GB. Above 32GB disables compressed oops.
10. **Fluentd memory buffering** — memory buffers without limits cause OOM. Use file-based buffers for reliability.

---

## 中文版本

### 使用场景

- 为微服务搭建集中式日志系统
- 在 ELK Stack 和 Grafana Loki 之间做选择
- 配置 Fluentd、Fluent Bit 或 Vector 日志收集
- 编写 LogQL（Loki）或 KQL/Lucene（Elasticsearch）查询
- 管理日志保留策略和存储成本

### 核心步骤

1. **Loki 部署** — Helm 部署 Loki，配置 S3 存储、TSDB schema、30 天保留期和速率限制
2. **Fluent Bit DaemonSet** — 配置 tail input 采集容器日志，kubernetes filter 添加元数据，输出到 Loki
3. **Vector 现代替代** — 使用 Rust 编写的 Vector 替代 Fluentd，支持 JSON 解析、过滤、元数据丰富
4. **LogQL 查询** — 按 namespace/container 过滤，JSON 解析，错误率统计，Top-K 分析
5. **日志告警** — 配置 Loki Ruler 基于日志内容触发高错误率和 OOMKilled 告警

### 模板说明

- Loki Helm values — read/write/gateway 分离架构，S3 存储，速率限制配置
- Fluent Bit ConfigMap — 完整的 input/filter/output 配置，包含 CRI parser
- Vector 配置 — Kubernetes 日志采集、JSON 解析、过滤、Loki 输出的 TOML 配置
- LogQL 示例 — 基础查询、JSON 解析、pattern 解析、错误率统计、unwrap 聚合

### 常见陷阱

1. **生产环境使用 DEBUG 日志** — DEBUG 日志产生 10 倍数据量，生产环境设为 INFO/WARN
2. **无日志关联** — 没有 request_id/trace_id 时无法跨服务追踪请求
3. **Elasticsearch 集群过载** — 无 ILM 时高流量日志索引无限增长，磁盘满 = 集群 red
4. **Loki 高基数标签** — 将 `user_id` 作为标签索引会导致 Loki 内存爆炸
5. **Grep 式告警不精确** — `|= "error"` 会匹配 "error-free"，使用 `| json | level="error"` 精确过滤
