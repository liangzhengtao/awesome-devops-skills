# Kubernetes Essentials / Kubernetes 核心技能

> Core K8s resources, RBAC, networking, troubleshooting, and production patterns for running workloads reliably at scale.

## When to Use / 何时使用

Use this skill when:

- Deploying applications to Kubernetes for the first time
- Debugging pod crashes, networking issues, or resource problems
- Configuring RBAC, network policies, and pod security
- Setting up horizontal pod autoscaling (HPA) and vertical pod autoscaling (VPA)
- Implementing ConfigMaps, Secrets, and persistent storage
- Understanding Kubernetes networking (Services, Ingress, DNS)
- Performing cluster upgrades and node maintenance

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────┐
│                   Kubernetes Cluster                           │
│                                                               │
│  Control Plane          Worker Nodes                          │
│  ┌──────────────┐       ┌─────────────────────────────┐     │
│  │ API Server   │       │ Node 1                      │     │
│  │ etcd         │       │  ├── Pod (app-v2) ─┐        │     │
│  │ Scheduler    │       │  ├── Pod (app-v1) ─┤ Service │     │
│  │ Controller   │       │  └── Pod (app-v1) ─┘        │     │
│  │ Manager      │       ├─────────────────────────────┤     │
│  └──────────────┘       │ Node 2                      │     │
│                         │  ├── Pod (app-v1) ─┐        │     │
│  Networking             │  ├── Pod (worker)   │ Ingress│     │
│  ┌──────────────┐       │  └── Pod (cache)  ─┘        │     │
│  │ CoreDNS      │       └─────────────────────────────┘     │
│  │ kube-proxy   │                                            │
│  │ CNI (Cilium) │       Storage                              │
│  └──────────────┘       ┌──────────────┐                    │
│                         │ PV / PVC     │                    │
│  Ingress                │ StorageClass │                    │
│  ┌──────────────┐       └──────────────┘                    │
│  │ Ingress Ctrl │                                            │
│  │ cert-manager │                                            │
│  └──────────────┘                                            │
└──────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Production Deployment with All Proven Patterns

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  namespace: production
  labels:
    app.kubernetes.io/name: my-app
    app.kubernetes.io/version: "2.0.0"
    app.kubernetes.io/component: api
    app.kubernetes.io/part-of: my-platform
spec:
  replicas: 3
  revisionHistoryLimit: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app.kubernetes.io/name: my-app
  template:
    metadata:
      labels:
        app.kubernetes.io/name: my-app
        app.kubernetes.io/version: "2.0.0"
    spec:
      serviceAccountName: my-app
      automountServiceAccountToken: false
      terminationGracePeriodSeconds: 60
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
        seccompProfile:
          type: RuntimeDefault
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: my-app
      containers:
        - name: app
          image: registry.example.com/my-app:2.0.0@sha256:abc123
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
          env:
            - name: NODE_ENV
              value: production
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: my-app-db
                  key: password
          envFrom:
            - configMapRef:
                name: my-app-config
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: [ALL]
          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 15
            periodSeconds: 20
            timeoutSeconds: 5
            failureThreshold: 3
          startupProbe:
            httpGet:
              path: /health/start
              port: http
            failureThreshold: 30
            periodSeconds: 2
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: app-config
              mountPath: /app/config
              readOnly: true
      volumes:
        - name: tmp
          emptyDir:
            sizeLimit: 100Mi
        - name: app-config
          configMap:
            name: my-app-files
```

### 2. Service and Ingress

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  namespace: production
spec:
  selector:
    app.kubernetes.io/name: my-app
  ports:
    - name: http
      port: 80
      targetPort: http
  type: ClusterIP

---
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app
  namespace: production
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
spec:
  ingressClassName: nginx
  tls:
    - hosts: [api.example.com]
      secretName: my-app-tls
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app
                port:
                  name: http
```

### 3. RBAC Configuration

```yaml
# rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
  namespace: production

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: my-app-secrets-reader
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["my-app-db", "my-app-redis"]
    verbs: ["get", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-app-secrets-reader
  namespace: production
subjects:
  - kind: ServiceAccount
    name: my-app
    namespace: production
roleRef:
  kind: Role
  name: my-app-secrets-reader
  apiGroup: rbac.authorization.k8s.io
```

### 4. Horizontal Pod Autoscaler

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 20
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
        - type: Pods
          value: 2
          periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "1000"
```

### 5. Network Policy

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: my-app-netpol
  namespace: production
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: my-app
  policyTypes: [Ingress, Egress]
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - port: 8080
          protocol: TCP
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: database
      ports:
        - port: 5432
          protocol: TCP
    - to:  # Allow DNS
        - namespaceSelector: {}
      ports:
        - port: 53
          protocol: UDP
        - port: 53
          protocol: TCP
```

### 6. Troubleshooting Commands

```bash
# ── Pod Debugging ──────────────────────────────────
# Check pod status and events
kubectl get pods -n production -o wide
kubectl describe pod my-app-xyz -n production
kubectl logs my-app-xyz -n production --tail=100 --previous

# Ephemeral debug container (K8s 1.25+)
kubectl debug -it my-app-xyz -n production \
  --image=busybox:1.36 --target=app

# Check resource usage
kubectl top pods -n production --sort-by=memory
kubectl top nodes

# ── Networking Debugging ──────────────────────────
# DNS resolution test
kubectl run dns-test --image=busybox:1.36 --rm -it -- \
  nslookup my-app.production.svc.cluster.local

# Service endpoint verification
kubectl get endpoints my-app -n production
kubectl get ingress -n production

# Port forward for local testing
kubectl port-forward svc/my-app 8080:80 -n production

# ── Cluster Health ────────────────────────────────
kubectl get nodes -o wide
kubectl cluster-info
kubectl get events -n production --sort-by='.lastTimestamp' | tail -20

# Check certificate expiry
kubectl get certificates -n production
```

## Proven Patterns / 最佳实践

1. **Use namespaces for isolation** — separate environments (staging, production) and teams into namespaces with ResourceQuotas.
2. **Set resource requests AND limits** — requests for scheduling, limits for protection. Never skip requests; pods get low QoS class.
3. **Use `topologySpreadConstraints`** — distribute pods across zones/nodes for high availability. Don't rely on `podAntiAffinity` alone.
4. **Implement PodDisruptionBudgets** — `minAvailable: 2` ensures voluntary disruptions (node drain) maintain availability.
5. **Use `automountServiceAccountToken: false`** — disable unless the pod needs K8s API access. Reduces attack surface.
6. **Pin images by digest** — `image: repo/app@sha256:abc123` prevents tag mutation attacks and ensures reproducibility.
7. **Use startup probes for slow apps** — startup probes run before liveness probes, preventing slow-starting pods from being killed.
8. **Label everything** — use the `app.kubernetes.io/*` recommended labels for service mesh, monitoring, and cost allocation.
9. **Use NetworkPolicies** — default-allow is dangerous. Deny all ingress/egress, then explicitly allow what's needed.
10. **Monitor resource usage** — use `kubectl top` and metrics-server. Set alerts before hitting resource limits.

## Pitfalls / 常见陷阱

1. **No resource requests** — pods without requests get BestEffort QoS and are evicted first during node pressure.
2. **Liveness probe too aggressive** — a liveness probe with `failureThreshold: 1` and `periodSeconds: 1` restarts pods constantly. Use `failureThreshold: 3` and `periodSeconds: 10+`.
3. **Missing `terminationGracePeriodSeconds`** — the default 30s may not be enough for long-running requests. Match it to your shutdown drain time.
4. **ConfigMap/Secret not reloading** — mounted ConfigMaps don't auto-reload. Use tools like Reloader or restart deployments on change.
5. **Service selector mismatch** — if the Service selector doesn't match the Pod labels, the Service has no endpoints. Use `kubectl get endpoints` to verify.
6. **Ingress path type confusion** — `pathType: Exact` only matches exactly. Use `pathType: Prefix` for sub-paths.
7. **DNS search domain issues** — `my-app` resolves within the same namespace; cross-namespace requires `my-app.other-ns.svc.cluster.local`.
8. **OOMKilled but not detected** — `kubectl describe pod` shows OOMKilled in the last state, but `kubectl logs` won't show why. Check events.
9. **PVC Pending state** — PersistentVolumeClaims stay Pending when no matching PV or StorageClass exists. Check `kubectl describe pvc`.
10. **Cluster DNS overload** — every pod makes DNS queries. CoreDNS needs proper resource limits and HPA configured for large clusters.

---

## 中文版本

### 使用场景

- 首次将应用部署到 Kubernetes
- 调试 Pod 崩溃、网络问题或资源问题
- 配置 RBAC、NetworkPolicy 和 Pod 安全策略
- 设置 HPA 和 VPA 自动扩缩容
- 理解 Kubernetes 网络（Service、Ingress、DNS）

### 核心步骤

1. **编写生产级 Deployment** — 设置 resource requests/limits、securityContext（non-root、readOnlyRootFilesystem）、topologySpreadConstraints
2. **配置 Service + Ingress** — ClusterIP Service + Nginx Ingress + cert-manager 自动 TLS
3. **RBAC 最小权限** — 创建专用 ServiceAccount，仅授予所需的 Secret 读取权限
4. **HPA 自动扩缩** — 基于 CPU/内存/自定义指标（http_requests_per_second）自动扩缩
5. **NetworkPolicy 隔离** — 默认拒绝所有流量，显式允许 ingress-nginx 和数据库访问

### 模板说明

- Production Deployment — 包含所有最佳实践的完整 Deployment YAML
- Service + Ingress — 配合 cert-manager 和 rate-limit 的 Ingress 配置
- RBAC — ServiceAccount、Role、RoleBinding 最小权限配置
- HPA — 基于多指标的自动扩缩容，包含扩缩行为控制

### 常见陷阱

1. **无 resource requests** — 没有 requests 的 Pod 获得 BestEffort QoS，节点压力时最先被驱逐
2. **Liveness probe 过于激进** — `failureThreshold: 1` + `periodSeconds: 1` 会不断重启 Pod
3. **ConfigMap/Secret 不自动重载** — 挂载的 ConfigMap 不会自动刷新，需使用 Reloader 或重启 Deployment
4. **Service selector 不匹配** — selector 与 Pod label 不匹配时 Service 无 endpoint，用 `kubectl get endpoints` 验证
5. **OOMKilled 难以发现** — `kubectl describe pod` 显示 OOMKilled 但 `kubectl logs` 不显示原因，需检查 events
