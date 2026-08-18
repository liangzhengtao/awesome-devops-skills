[中文版](README.zh.md)

# Awesome DevOps Skills

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

> **Stop firefighting. 12 AI skills to automate your infrastructure.**

A curated collection of production-ready AI skills for DevOps and infrastructure automation. Each skill file is a self-contained knowledge module that an AI agent can load to help you build, deploy, monitor, and operate production systems.

---

## Table of Contents

- [Skills Overview](#skills-overview)
- [Quick Start](#quick-start)
- [CI/CD](#cicd)
- [Containerization](#containerization)
- [Cloud](#cloud)
- [Monitoring](#monitoring)
- [Contributing](#contributing)
- [See Also](#see-also)

---

## Skills Overview

| # | Category | Skill | Description |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md) | Matrix builds, OIDC, reusable workflows, caching |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Runners, DAG pipelines, environments, security scanning |
| 3 | CI/CD | [Deployment Strategies](skills/CI-CD/deployment-strategies.md) | Blue-green, canary, rolling, Argo Rollouts |
| 4 | Containerization | [Docker Production](skills/容器化/docker-production.md) | Multi-stage builds, security hardening, image optimization |
| 5 | Containerization | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, networking, troubleshooting |
| 6 | Containerization | [Helm Charts](skills/容器化/helm-charts.md) | Chart development, templating, testing, repositories |
| 7 | Cloud | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, cost optimization |
| 8 | Cloud | [Terraform IaC](skills/云服务/terraform-iac.md) | Modules, state management, workspaces, CI/CD integration |
| 9 | Cloud | [Serverless Patterns](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | Monitoring | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporters, PromQL, dashboards, SLO-based alerts |
| 11 | Monitoring | [Logging Stack](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, retention |
| 12 | Monitoring | [Incident Response](skills/监控告警/incident-response.md) | On-call, runbooks, post-mortems, escalation |

---

## Quick Start

### For AI Agent Users

1. **Choose a skill** from the table above based on your task
2. **Load the skill file** into your AI agent's context
3. **Apply the templates** — each skill contains production-ready code

### For Developers

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### Skill File Structure

Each skill follows a consistent format:

```
┌─────────────────────────────────┐
│  When to Use                    │  ← Clear trigger conditions
│  Architecture                   │  ← ASCII diagrams
│  Code Templates                 │  ← YAML/HCL/Dockerfile
│  Proven Patterns                │  ← Battle-tested recommendations
│  Pitfalls                       │  ← Common mistakes to avoid
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md)
Advanced patterns for GitHub Actions: matrix builds, OIDC authentication, reusable workflows, caching strategies, self-hosted runners, and semantic release automation.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
Production GitLab pipelines: DAG optimization, autoscaling runners, review apps, security scanning templates, and multi-environment deployments.

### [Deployment Strategies](skills/CI-CD/deployment-strategies.md)
Zero-downtime deployment patterns: blue-green with Argo Rollouts, canary with traffic splitting and automated analysis, rolling updates with proper health checks.

---

## Containerization

### [Docker Production](skills/容器化/docker-production.md)
Production Docker patterns: multi-stage builds (45MB images), non-root containers, security scanning with Trivy, multi-architecture builds, and Docker Compose for production.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
K8s production patterns: deployments with topology spread, RBAC least-privilege, HPA with custom metrics, NetworkPolicies, and complete troubleshooting commands.

### [Helm Charts](skills/容器化/helm-charts.md)
Helm proven patterns: modular chart structure, template helpers, multi-environment values, `--atomic` deployments, chart testing, and OCI registry publishing.

---

## Cloud

### [AWS Essentials](skills/云服务/aws-essentials.md)
AWS production architecture: VPC with multi-AZ, IAM least-privilege with OIDC, ECS Fargate services, Aurora Serverless, S3 security, and cost optimization with budgets.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform at scale: reusable modules, remote state with locking, environment separation, CI/CD with plan/apply, Terratest, and state migration strategies.

### [Serverless Patterns](skills/云服务/serverless-patterns.md)
Event-driven serverless: Lambda with API Gateway, Step Functions workflows, EventBridge rules, DynamoDB single-table design, cold start optimization, and SAM templates.

---

## Monitoring

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Full observability stack: Prometheus configuration, recording rules, SLO-based alerting, Grafana dashboards with variables, Alertmanager routing, and Thanos for long-term storage.

### [Logging Stack](skills/监控告警/logging-stack.md)
Centralized logging: Grafana Loki deployment, Fluent Bit/Vector log shipping, LogQL queries, Elasticsearch ILM policies, structured logging, and log-based alerts.

### [Incident Response](skills/监控告警/incident-response.md)
Incident management: PagerDuty escalation policies, runbook templates, post-mortem format, severity definitions, communication templates, and SLO error budget tracking.

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

**How to contribute:**

1. Fork the repository
2. Create a feature branch
3. Add or improve a skill file
4. Submit a Pull Request

**Skill requirements:**
- 150+ lines per file
- Must include all 5 sections
- Production-ready code templates

---

## See Also

Other awesome collections you might find useful:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Self-hosted software
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetes resources
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Docker ecosystem
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraform modules and tools
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheus ecosystem
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWS resources
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Site Reliability Engineering
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm charts and tools
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CD pipeline tools
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafana dashboards and plugins
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Incident management resources
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Serverless frameworks and tools

---

## License

[MIT License](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Made with care for the DevOps community<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Star this repo</a> if you find it useful
</p>
