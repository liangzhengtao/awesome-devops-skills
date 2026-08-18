# Prometheus + Grafana Monitoring / Prometheus + Grafana 监控体系

> Complete observability stack with Prometheus exporters, PromQL, Grafana dashboards, alerting rules, and high-availability setup.

## When to Use / 何时使用

Use this skill when:

- Setting up application and infrastructure monitoring from scratch
- Writing PromQL queries for dashboards and alerts
- Configuring Prometheus exporters for various services (Node, MySQL, Redis, Nginx)
- Building Grafana dashboards with panels, variables, and annotations
- Designing alerting rules with proper thresholds and escalation
- Implementing high-availability Prometheus with Thanos or Mimir
- Migrating from other monitoring systems (Nagios, Zabbix, Datadog)

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Monitoring Architecture                         │
│                                                                   │
│  Applications ──► /metrics ──► Prometheus ──► Grafana            │
│       │                           │               │               │
│  Node Exporter ──────────────────┤           Dashboards          │
│  cAdvisor ───────────────────────┤           - SLOs              │
│  kube-state-metrics ─────────────┤           - Infrastructure    │
│  Redis Exporter ─────────────────┤           - Application       │
│  MySQL Exporter ─────────────────┤                               │
│       │                           │                               │
│       │                    Alertmanager ──► PagerDuty             │
│       │                           │        ► Slack                │
│       │                           │        ► Email                │
│       │                           │                               │
│       │                    Thanos/Mimir ──► Long-term Storage     │
│       │                                        (S3/GCS)           │
│  Pushgateway ◄── Batch Jobs                                       │
│       │                                                           │
│  Blackbox Exporter ──► External endpoints                        │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Prometheus Configuration (Helm Values)

```yaml
# prometheus-values.yaml
server:
  retention: 15d
  persistentVolume:
    size: 100Gi
    storageClass: gp3

  global:
    scrape_interval: 15s
    evaluation_interval: 15s
    scrape_timeout: 10s

  extraArgs:
    web.enable-lifecycle: true        # POST /-/reload to reload config
    storage.tsdb.min-block-duration: 2h

  resources:
    requests:
      cpu: 500m
      memory: 2Gi
    limits:
      cpu: "2"
      memory: 4Gi

alertmanager:
  enabled: true
  config:
    global:
      resolve_timeout: 5m
      slack_api_url: "${SLACK_WEBHOOK_URL}"
    route:
      group_by: ['alertname', 'namespace']
      group_wait: 30s
      group_interval: 5m
      repeat_interval: 4h
      receiver: 'slack-critical'
      routes:
        - match:
            severity: critical
          receiver: 'pagerduty'
        - match:
            severity: warning
          receiver: 'slack-warnings'
    receivers:
      - name: 'slack-critical'
        slack_configs:
          - channel: '#alerts-critical'
            send_resolved: true
            title: '{{ .GroupLabels.alertname }}'
            text: >-
              {{ range .Alerts }}
              *{{ .Annotations.summary }}*
              {{ .Annotations.description }}
              {{ end }}
      - name: 'pagerduty'
        pagerduty_configs:
          - service_key: "${PAGERDUTY_KEY}"
            severity: '{{ .GroupLabels.severity }}'
      - name: 'slack-warnings'
        slack_configs:
          - channel: '#alerts-warning'
            send_resolved: true

nodeExporter:
  enabled: true

kubeStateMetrics:
  enabled: true

# Additional scrape configs
extraScrapeConfigs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_port, __meta_kubernetes_pod_ip]
        action: replace
        regex: (\d+);(([A-Fa-f0-9]{1,4}::?){1,7}[A-Fa-f0-9]{1,4}|(\d{1,3}\.){3}\d{1,3})
        replacement: $$2:$$1
        target_label: __address__
```

### 2. Recording Rules (Performance Optimization)

```yaml
# recording-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-recording-rules
  namespace: monitoring
spec:
  groups:
    - name: app.rules
      interval: 30s
      rules:
        # Request rate (5-minute window)
        - record: app:http_requests:rate5m
          expr: |
            sum(rate(http_requests_total[5m])) by (service, method, status)

        # Error rate
        - record: app:http_errors:ratio5m
          expr: |
            sum(rate(http_requests_total{status=~"5.."}[5m])) by (service)
            /
            sum(rate(http_requests_total[5m])) by (service)

        # P99 latency
        - record: app:http_latency:p99_5m
          expr: |
            histogram_quantile(0.99,
              sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
            )

        # P95 latency
        - record: app:http_latency:p95_5m
          expr: |
            histogram_quantile(0.95,
              sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service)
            )

        # Apdex score
        - record: app:http_apdex:ratio5m
          expr: |
            (
              sum(rate(http_request_duration_seconds_bucket{le="0.1"}[5m])) by (service)
              + sum(rate(http_request_duration_seconds_bucket{le="0.5"}[5m])) by (service)
            ) / 2
            /
            sum(rate(http_request_duration_seconds_count[5m])) by (service)
```

### 3. Alerting Rules (SLO-Based)

```yaml
# alerting-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: app-alerts
  namespace: monitoring
spec:
  groups:
    - name: slo.alerts
      rules:
        # ── Availability SLO ──────────────────────────
        - alert: HighErrorRate
          expr: app:http_errors:ratio5m > 0.01
          for: 5m
          labels:
            severity: critical
            team: platform
          annotations:
            summary: "Error rate above 1% for {{ $labels.service }}"
            description: "Error rate is {{ $value | humanizePercentage }} (threshold: 1%)"
            runbook_url: "https://wiki.internal/runbooks/high-error-rate"

        - alert: ErrorRateSLOBurn
          expr: |
            app:http_errors:ratio5m > 0.001
            and
            predict_linear(app:http_errors:ratio5m[1h], 24*3600) > 0.01
          for: 15m
          labels:
            severity: warning
          annotations:
            summary: "SLO burn rate will exceed budget within 24h"

        # ── Latency SLO ───────────────────────────────
        - alert: HighP99Latency
          expr: app:http_latency:p99_5m > 2
          for: 10m
          labels:
            severity: warning
          annotations:
            summary: "P99 latency above 2s for {{ $labels.service }}"
            description: "P99 latency is {{ $value }}s"

        # ── Infrastructure ────────────────────────────
        - alert: HighCPUUsage
          expr: |
            100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
          for: 15m
          labels:
            severity: warning
          annotations:
            summary: "CPU usage above 85% on {{ $labels.instance }}"

        - alert: HighMemoryUsage
          expr: |
            (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
          for: 10m
          labels:
            severity: critical
          annotations:
            summary: "Memory usage above 90% on {{ $labels.instance }}"

        - alert: DiskSpaceLow
          expr: |
            (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "Disk space below 15% on {{ $labels.instance }}"

        - alert: PodCrashLooping
          expr: |
            rate(kube_pod_container_status_restarts_total[15m]) * 60 * 15 > 0
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "Pod {{ $labels.pod }} is crash looping"

        - alert: PersistentVolumeAlmostFull
          expr: |
            kubelet_volume_stats_available_bytes / kubelet_volume_stats_capacity_bytes < 0.10
          for: 5m
          labels:
            severity: critical
          annotations:
            summary: "PV {{ $labels.persistentvolumeclaim }} is almost full"
```

### 4. Grafana Dashboard JSON (Core Panels)

```json
{
  "dashboard": {
    "title": "Application Overview",
    "tags": ["app", "production"],
    "templating": {
      "list": [
        {
          "name": "namespace",
          "type": "query",
          "query": "label_values(kube_pod_info, namespace)",
          "refresh": 2
        },
        {
          "name": "service",
          "type": "query",
          "query": "label_values(http_requests_total{namespace=\"$namespace\"}, service)",
          "refresh": 2
        }
      ]
    },
    "panels": [
      {
        "title": "Request Rate",
        "type": "timeseries",
        "targets": [{
          "expr": "sum(rate(http_requests_total{namespace=\"$namespace\", service=\"$service\"}[5m])) by (status)",
          "legendFormat": "{{status}}"
        }]
      },
      {
        "title": "Error Rate",
        "type": "gauge",
        "targets": [{
          "expr": "app:http_errors:ratio5m{namespace=\"$namespace\", service=\"$service\"}"
        }],
        "fieldConfig": {
          "defaults": {
            "thresholds": {
              "steps": [
                {"value": 0, "color": "green"},
                {"value": 0.001, "color": "yellow"},
                {"value": 0.01, "color": "red"}
              ]
            }
          }
        }
      },
      {
        "title": "Latency Distribution",
        "type": "timeseries",
        "targets": [
          {"expr": "app:http_latency:p99_5m{namespace=\"$namespace\", service=\"$service\"}", "legendFormat": "p99"},
          {"expr": "app:http_latency:p95_5m{namespace=\"$namespace\", service=\"$service\"}", "legendFormat": "p95"},
          {"expr": "histogram_quantile(0.50, sum(rate(http_request_duration_seconds_bucket{namespace=\"$namespace\", service=\"$service\"}[5m])) by (le))", "legendFormat": "p50"}
        ]
      }
    ]
  }
}
```

### 5. Docker Compose Monitoring Stack

```yaml
# docker-compose.monitoring.yml
services:
  prometheus:
    image: prom/prometheus:v2.51.0
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/rules:/etc/prometheus/rules
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.retention.time=30d'
      - '--web.enable-lifecycle'
    ports: ["9090:9090"]

  grafana:
    image: grafana/grafana:10.4.0
    environment:
      GF_SECURITY_ADMIN_PASSWORD: "${GRAFANA_PASSWORD}"
      GF_INSTALL_PLUGINS: grafana-clock-panel,grafana-piechart-panel
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
      - ./grafana/dashboards:/var/lib/grafana/dashboards
    ports: ["3000:3000"]
    depends_on: [prometheus]

  alertmanager:
    image: prom/alertmanager:v0.27.0
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'
    ports: ["9093:9093"]

  node-exporter:
    image: prom/node-exporter:v1.7.0
    command:
      - '--path.rootfs=/host'
    volumes:
      - '/:/host:ro,rslave'
    network_mode: host

volumes:
  prometheus-data:
  grafana-data:
```

## Best Practices / 最佳实践

1. **Use recording rules for expensive queries** — pre-compute P99, error rates, and Apdex scores. Dashboards load 10x faster.
2. **Follow RED/USE methods** — Rate, Errors, Duration for services; Utilization, Saturation, Errors for infrastructure.
3. **Set SLO-based alerts** — alert on error budget burn rate, not arbitrary thresholds. Reduces alert fatigue.
4. **Use `for` duration wisely** — `for: 5m` avoids flapping alerts. Critical alerts can have shorter durations.
5. **Label consistently** — use `service`, `namespace`, `environment`, `team` labels across all metrics.
6. **Dashboard variables** — use Grafana template variables for namespace, service, and instance filtering. One dashboard serves all environments.
7. **Monitor the monitoring** — alert on Prometheus scrape failures, storage filling up, and Alertmanager delivery failures.
8. **Use Thanos/Mimir for long-term storage** — Prometheus local storage is limited to weeks. Thanos adds years of retention on S3.
9. **Rate over raw counters** — always use `rate()` or `increase()` with counters. Never graph raw counter values.
10. **Cardinality management** — high-cardinality labels (user_id, request_id) explode storage. Use exemplars instead.

## Pitfalls / 常见陷阱

1. **High cardinality explosion** — a label with 100K values × 100 metrics = 10M time series. Monitor `prometheus_tsdb_head_series` for growth.
2. **Missing `for` in alert rules** — without `for`, a momentary spike triggers an alert. Use `for: 5m` minimum for most alerts.
3. **Stale metrics after deployment** — old pod metrics linger until the stale marker (5m). Don't alert on instant metrics after deploy.
4. **Recording rules too granular** — recording by every label defeats caching. Record by the labels your dashboards actually use.
5. **Prometheus scrape timeout** — if scrape takes longer than `scrape_timeout`, metrics are dropped. Increase timeout for slow exporters.
6. **Grafana dashboard over-querying** — 50 panels each running independent queries overwhelm Prometheus. Use recording rules.
7. **Alertmanager grouping** — too broad `group_by` batches unrelated alerts. Too narrow sends duplicate pages.
8. **No SLO error budget** — without error budgets, teams either ignore all alerts or burn out on pages. Define 99.9% SLO = 43.8 min/month downtime budget.
9. **Rate interval too short** — `rate(metric[1m])` is noisy. Use `rate(metric[5m])` for stability.
10. **Missing kube-state-metrics** — pod, deployment, and node metadata requires kube-state-metrics. Without it, K8s resource metrics are incomplete.

---

## 中文版本

### 使用场景

- 从零搭建应用和基础设施监控
- 编写 PromQL 查询用于仪表盘和告警
- 配置各种服务的 Prometheus exporter（Node、MySQL、Redis、Nginx）
- 设计基于 SLO 的告警规则和升级策略
- 实现高可用 Prometheus（Thanos/Mimir）

### 核心步骤

1. **Prometheus 配置** — 设置 scrape_interval、retention、持久化存储，配置 Kubernetes pod 自动发现
2. **Recording Rules** — 预计算请求速率、错误率、P99 延迟等高频查询，仪表盘加载速度提升 10x
3. **SLO 告警规则** — 基于错误预算燃烧率告警，而非任意阈值，减少告警疲劳
4. **Alertmanager 路由** — 按 severity 路由到 PagerDuty（critical）和 Slack（warning），配置分组和抑制
5. **Grafana 仪表盘** — 使用模板变量（namespace、service）实现一个仪表盘服务所有环境

### 模板说明

- Prometheus Helm values — 包含 server、alertmanager、node-exporter、kube-state-metrics 的完整配置
- Recording Rules — 请求速率、错误率、P99/P95 延迟、Apdex 分数的预计算规则
- Alerting Rules — 高错误率、SLO 燃烧率、高 CPU/内存、Pod crash loop、PV 空间不足等告警
- Grafana Dashboard JSON — 请求速率、错误率、延迟分布的核心面板

### 常见陷阱

1. **高基数标签爆炸** — `user_id` 等标签有 10 万个值 × 100 个指标 = 1000 万时间序列
2. **告警规则缺少 `for`** — 没有 `for` 时瞬时尖峰会触发告警，大多数告警至少 `for: 5m`
3. **Recording rules 过于细粒度** — 按每个标签 recording 会失去缓存优势，只按仪表盘实际使用的标签
4. **Prometheus scrape 超时** — scrape 超过 `scrape_timeout` 时指标被丢弃，慢 exporter 需增加超时
5. **无 SLO 错误预算** — 没有错误预算时团队要么忽略所有告警要么被 page 疲劳淹没
