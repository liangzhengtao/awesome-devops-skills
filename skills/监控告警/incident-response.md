# Incident Response and On-Call / 故障响应与值班

> On-call workflows, runbooks, escalation policies, post-mortem templates, and incident management proven patterns.

## When to Use / 何时使用

Use this skill when:

- Setting up on-call rotations and escalation policies
- Creating runbooks for common infrastructure failures
- Designing incident response workflows and communication templates
- Writing post-mortems and tracking action items
- Configuring PagerDuty, OpsGenie, or Grafana OnCall
- Building incident dashboards and SLO tracking
- Training teams on incident management procedures
- Implementing blameless post-mortem culture

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Incident Response Pipeline                     │
│                                                                   │
│  Detection           Response              Resolution            │
│  ┌─────────┐        ┌──────────┐         ┌───────────┐          │
│  │ Monitor │──alert─►│Alertmgr  │─page──►│ On-Call   │          │
│  │         │        │          │         │ Engineer  │          │
│  │ Synth.  │────────►│          │─slack──►│           │          │
│  │ Checks  │        └──────────┘         └─────┬─────┘          │
│  └─────────┘                                   │                 │
│                                                ▼                 │
│  Communication       Post-Mortem          ┌───────────┐         │
│  ┌─────────┐        ┌──────────┐         │ Triage &  │         │
│  │ Status  │        │ Template │◄────────│ Mitigate  │         │
│  │ Page    │        │          │         └───────────┘         │
│  │         │        │ Action   │                                │
│  │ War     │        │ Items    │         ┌───────────┐         │
│  │ Room    │        └──────────┘         │ Resolve & │         │
│  └─────────┘                             │ Verify    │         │
│                                          └───────────┘         │
│                                                                   │
│  Severity:  SEV1 (< 15 min)  SEV2 (< 1 hr)  SEV3 (< 4 hr)     │
│  Tools:     PagerDuty / Grafana OnCall / OpsGenie              │
│  Comms:     Slack #incidents, StatusPage, Email                 │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. PagerDuty Escalation Policy (Terraform)

```hcl
# pagerduty.tf
resource "pagerduty_team" "platform" {
  name        = "Platform Engineering"
  description = "Infrastructure and platform team"
}

resource "pagerduty_user" "oncall_engineers" {
  for_each = {
    alice = { name = "Alice Chen",   email = "alice@example.com" }
    bob   = { name = "Bob Smith",    email = "bob@example.com" }
    carol = { name = "Carol Davis",  email = "carol@example.com" }
    dave  = { name = "Dave Wilson",  email = "dave@example.com" }
  }
  name  = each.value.name
  email = each.value.email
}

resource "pagerduty_schedule" "primary" {
  name      = "Primary On-Call"
  time_zone = "America/New_York"

  layer {
    name                         = "Weekly Rotation"
    rotation_turn_length_seconds = 604800  # 7 days
    rotation_virtual_start       = "2026-01-05T09:00:00-05:00"

    users = [
      pagerduty_user.oncall_engineers["alice"].id,
      pagerduty_user.oncall_engineers["bob"].id,
      pagerduty_user.oncall_engineers["carol"].id,
      pagerduty_user.oncall_engineers["dave"].id,
    ]
  }

  restriction {
    type              = "during"
    start_time_of_day = "09:00:00"
    end_time_of_day   = "17:00:00"
    duration_seconds  = 28800
  }
}

resource "pagerduty_schedule" "secondary" {
  name      = "Secondary On-Call"
  time_zone = "America/New_York"
  # Similar config with different rotation
}

resource "pagerduty_escalation_policy" "platform" {
  name      = "Platform Engineering Escalation"
  num_loops = 2

  rule {
    escalation_delay_in_minutes = 10
    target {
      type = "schedule_reference"
      id   = pagerduty_schedule.primary.id
    }
  }

  rule {
    escalation_delay_in_minutes = 15
    target {
      type = "schedule_reference"
      id   = pagerduty_schedule.secondary.id
    }
  }

  rule {
    escalation_delay_in_minutes = 20
    target {
      type = "user_reference"
      id   = pagerduty_user.oncall_engineers["alice"].id  # Engineering Manager
    }
  }
}

# ── Services and Alert Rules ─────────────────────────
resource "pagerduty_service" "api" {
  name              = "Production API"
  escalation_policy = pagerduty_escalation_policy.platform.id
  alert_creation    = "create_alerts_and_incidents"

  auto_pause_transient_alerts = true

  incident_urgency_rule {
    type    = "use_support_hours"
    during_support_hours {
      type    = "constant"
      urgency = "high"
    }
    outside_support_hours {
      type    = "constant"
      urgency = "low"
    }
  }
}
```

### 2. Runbook Template

```markdown
# Runbook: High Error Rate on Production API

## Alert
`HighErrorRate` — Error rate > 1% for 5 minutes on `api-server`

## Severity
SEV2 — Customer-facing impact, requires immediate attention

## Impact
- Users receiving 500 errors on API requests
- Affected services: web app, mobile app, partner integrations
- Estimated impact scope: Check error rate percentage

## Triage Steps (in order)

### Step 1: Confirm the Alert
```bash
# Check current error rate in Grafana
# Dashboard: Application Overview > Error Rate panel

# Or query Prometheus directly
promql 'app:http_errors:ratio5m{service="api-server"}'

# Check Loki for error patterns
logql '{namespace="production", container="api-server"} |= "error" | json | line_format "{{.error}}" | top 10'
```

### Step 2: Identify the Scope
```bash
# Is it all endpoints or specific ones?
promql 'sum(rate(http_requests_total{service="api-server", status=~"5.."}[5m])) by (path)'

# Is it all pods or specific instances?
promql 'sum(rate(http_requests_total{service="api-server", status=~"5.."}[5m])) by (pod)'

# Check recent deployments
kubectl rollout history deployment/api-server -n production

# Check pod status
kubectl get pods -n production -l app=api-server -o wide
```

### Step 3: Check Dependencies
```bash
# Database connectivity
kubectl exec -n production deploy/api-server -- pg_isready -h $DB_HOST

# Redis connectivity
kubectl exec -n production deploy/api-server -- redis-cli -h $REDIS_HOST ping

# External service health
curl -s https://external-api.example.com/health
```

### Step 4: Mitigation Options

**Option A: Rollback (if recent deployment)**
```bash
kubectl rollout undo deployment/api-server -n production
kubectl rollout status deployment/api-server -n production --timeout=120s
```

**Option B: Scale up (if load-related)**
```bash
kubectl scale deployment/api-server -n production --replicas=10
```

**Option C: Enable circuit breaker (if dependency failure)**
```bash
kubectl set env deployment/api-server -n production CIRCUIT_BREAKER_ENABLED=true
```

**Option D: Enable feature flag (if feature-related)**
```bash
# Disable the problematic feature via LaunchDarkly/similar
curl -X PATCH https://app.launchdarkly.com/api/v2/flags/my-app/disable-feature \
  -H "Authorization: $LD_API_KEY"
```

### Step 5: Verify Resolution
```bash
# Error rate should drop below threshold
promql 'app:http_errors:ratio5m{service="api-server"} < 0.01'

# Check customer reports
# Monitor #support slack channel for 30 minutes
```

## Escalation
- If unresolved in 15 minutes → page secondary on-call
- If unresolved in 30 minutes → page engineering manager
- If data integrity issue → page database team immediately

## Related Incidents
- Link to previous similar incidents
- Link to relevant post-mortems
```

### 3. Post-Mortem Template

```markdown
# Post-Mortem: [INCIDENT TITLE]

## Metadata
- **Date:** 2026-08-17
- **Duration:** 47 minutes (14:23 UTC — 15:10 UTC)
- **Severity:** SEV2
- **Incident Commander:** Alice Chen
- **Author:** Bob Smith
- **Status:** Published

## Executive Summary
On August 17, 2026, the production API experienced a 47-minute outage
affecting approximately 12,000 users. The root cause was a database
migration that added a NOT NULL column without a default value, causing
all INSERT queries to fail.

## Impact
- **Users affected:** ~12,000 (30% of traffic)
- **Revenue impact:** Estimated $8,400 in failed transactions
- **Duration:** 47 minutes
- **SLA impact:** Monthly error budget consumed: 62% (23.6 min remaining)

## Timeline (UTC)
| Time  | Event |
|-------|-------|
| 14:15 | Deployment #1847 starts rolling out with DB migration |
| 14:23 | Error rate exceeds 1% threshold, alert fires |
| 14:25 | On-call engineer acknowledges incident |
| 14:28 | Confirmed error rate at 15%, all inserts failing |
| 14:32 | Identified deployment #1847 as cause via rollout history |
| 14:35 | Initiated rollback of application deployment |
| 14:42 | Rollback complete, but DB migration still active |
| 14:45 | DBA manually applied DEFAULT value to column |
| 14:50 | Error rate dropping, monitoring recovery |
| 15:10 | Error rate returned to baseline, incident resolved |

## Root Cause
The migration script (`V42__add_user_score.sql`) added a `score` column
with `NOT NULL` constraint but no `DEFAULT` value. Existing code attempted
INSERTs without the `score` field, causing PostgreSQL to reject all writes.

## Contributing Factors
1. Migration was not reviewed by a DBA or platform engineer
2. No CI check for backward-compatible migrations
3. Staging database had different data volume than production
4. Rollback procedure did not include database migration rollback

## What Went Well
- Alert fired within 2 minutes of impact start
- On-call responded within 2 minutes
- Rollback of application was fast (7 minutes)
- Communication was clear in #incidents channel

## What Went Poorly
- Database migration was not reversible
- No pre-flight check for breaking migrations
- Rollback did not fully resolve (DB state persisted)
- 15 minutes elapsed between app rollback and DB fix

## Action Items
| # | Action | Owner | Priority | Due Date | Status |
|---|--------|-------|----------|----------|--------|
| 1 | Add migration CI check for NOT NULL without DEFAULT | Bob | P1 | 2026-08-24 | TODO |
| 2 | Create runbook for database migration rollback | Carol | P1 | 2026-08-24 | TODO |
| 3 | Implement `expand-contract` migration pattern | Alice | P2 | 2026-09-07 | TODO |
| 4 | Add staging data seeding to match production volume | Dave | P2 | 2026-09-07 | TODO |
| 5 | Review all pending migrations for similar issues | Bob | P1 | 2026-08-18 | TODO |

## Lessons Learned
1. **Database migrations are deployments too.** They need the same review, testing, and rollback procedures as code deployments.
2. **Backward compatibility is non-negotiable.** Every migration must be safe to run while old code is still deployed.
3. **Rollback must be complete.** If rollback doesn't restore the system fully, it's not a rollback — it's a partial mitigation.

## Links
- [Grafana Dashboard during incident](https://grafana.example.com/d/xxx?from=xxx&to=xxx)
- [Slack thread](https://example.slack.com/archives/C0XXXXXXX/p1234567890)
- [Deployment #1847](https://github.com/org/my-app/commit/abc123)
```

### 4. Incident Communication Templates

```markdown
# ── Initial Notification (within 5 minutes) ──
🚨 **INCIDENT DECLARED — SEV2**

**Service:** Production API
**Impact:** Users experiencing 500 errors on API requests
**Status:** Investigating
**Incident Commander:** @alice
**War Room:** #incidents-2026-08-17

We are investigating elevated error rates on the production API.
Updates will follow every 15 minutes.

---

# ── Status Update (every 15 minutes) ──
📋 **INCIDENT UPDATE — SEV2**

**Time:** 14:45 UTC (22 minutes in)
**Status:** Identified — Mitigating
**Current impact:** Error rate at 8%, down from 15%

**What we know:**
- Root cause identified as recent deployment
- Rollback in progress
- ETA for resolution: ~15 minutes

---

# ── Resolution ──
✅ **INCIDENT RESOLVED — SEV2**

**Duration:** 47 minutes (14:23 — 15:10 UTC)
**Root Cause:** Database migration incompatibility
**Resolution:** Application rollback + manual DB fix

Post-mortem will be published within 48 hours.
Thank you for your patience.
```

### 5. SLO Dashboard Queries

```promql
# ── Error Budget Calculation ────────────────────────
# Monthly error budget (99.9% SLO = 43.8 min/month)
# Current month's consumed budget

# Total minutes in current month
vector(time() - (time() - (day_of_month() - 1) * 24 * 60 * 60))

# Error budget remaining (minutes)
(
  0.001 * (time() - (time() - (day_of_month() - 1) * 24 * 60 * 60))
  -
  sum(increase(http_requests_total{status=~"5.."}[30d])) / sum(increase(http_requests_total[30d]))
    * (time() - (time() - (day_of_month() - 1) * 24 * 60 * 60))
) / 60

# Error budget burn rate (1 = burning at expected pace)
sum(rate(http_requests_total{status=~"5.."}[1h])) / sum(rate(http_requests_total[1h])) / 0.001
```

## Proven Patterns / 最佳实践

1. **Acknowledge alerts within 5 minutes** — unacknowledged alerts escalate. If you're investigating, acknowledge immediately.
2. **Declare severity early** — don't wait to assess full impact. Declare SEV1 if there's any chance of data loss or full outage.
3. **Assign an Incident Commander** — one person coordinates. Others execute. The IC doesn't debug; they direct.
4. **Communicate early and often** — initial notification within 5 minutes, updates every 15 minutes, resolution notice immediately.
5. **Mitigate first, root-cause later** — rollback, scale up, feature-flag off. Don't spend 30 minutes debugging when a rollback takes 2 minutes.
6. **Blameless post-mortems** — focus on systems and processes, not individuals. "The process allowed this" not "Bob pushed bad code."
7. **Action items must be owned** — every action item needs a name, priority, and due date. Unowned items never get done.
8. **Runbooks before incidents** — write runbooks during calm periods. Include exact commands, expected output, and decision trees.
9. **Practice incident response** — run game days quarterly. Inject failures, practice communication, measure response times.
10. **Track error budgets** — when error budget is below 50% for the month, freeze non-critical deployments.

## Pitfalls / 常见陷阱

1. **Alert fatigue** — too many non-actionable alerts cause engineers to ignore pages. Audit and prune monthly.
2. **No on-call handoff** — switching on-call without a handoff meeting means the new on-call doesn't know recent incidents or ongoing issues.
3. **Root cause too shallow** — "human error" is never the root cause. Dig into why the process allowed the error.
4. **No follow-up on action items** — post-mortem without tracking action items is theater. Review open items weekly.
5. **Single point of failure in on-call** — if only one person knows the system, burnout is inevitable. Cross-train aggressively.
6. **SEV1 without war room** — for SEV1, a dedicated Slack channel or video call is mandatory. Information scattered across DMs is lost.
7. **Rollback without verification** — rolling back and assuming it worked without checking error rates is dangerous. Always verify.
8. **Not preserving evidence** — pod logs, events, and metrics disappear after restarts. Capture them before mitigation actions.
9. **Skipping the post-mortem** — "we're too busy" means you'll repeat the incident. Every SEV1/SEV2 gets a post-mortem, no exceptions.
10. **Over-escalating** — paging the CTO for a SEV3 erodes trust. Respect severity definitions and escalation policies.

---

## 中文版本

### 使用场景

- 搭建值班轮换和升级策略
- 为常见基础设施故障创建 runbook
- 设计故障响应流程和沟通模板
- 编写事后复盘（post-mortem）并跟踪行动项
- 配置 PagerDuty、OpsGenie 或 Grafana OnCall
- 建立无指责的复盘文化

### 核心步骤

1. **值班配置** — 使用 Terraform 管理 PagerDuty 轮换排班、升级策略和告警规则
2. **Runbook 编写** — 为每个告警编写分步排查指南，包含确认告警、识别范围、检查依赖、缓解措施、验证恢复
3. **故障沟通模板** — 初始通知（5 分钟内）、状态更新（每 15 分钟）、解决通知三阶段模板
4. **事后复盘** — 包含时间线、根因分析、贡献因素、做得好/差的方面、带 owner 的行动项
5. **SLO 仪表盘** — 计算错误预算消耗和燃烧率，预算低于 50% 时冻结非关键部署

### 模板说明

- PagerDuty Terraform — 用户、排班、升级策略、服务和告警规则的完整配置
- Runbook 模板 — 高错误率的完整排查和缓解步骤（含具体命令）
- Post-mortem 模板 — 包含元数据、影响、时间线、根因、行动项的标准化复盘模板
- 沟通模板 — SEV2 故障的初始通知、状态更新、解决通知模板

### 常见陷阱

1. **告警疲劳** — 太多不可操作的告警导致工程师忽略 page，每月审计并精简告警
2. **无值班交接** — 切换值班时不做交接意味着新值班不了解近期故障和进行中的问题
3. **根因分析太浅** — "人为失误"不是根因，深挖为什么流程允许这个错误发生
4. **行动项无人跟踪** — 没有跟踪的复盘就是走过场，每周审查未完成行动项
5. **过度升级** — 为 SEV3 去 page CTO 会侵蚀信任，遵守严重性定义和升级策略
