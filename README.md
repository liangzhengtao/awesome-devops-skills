# Awesome DevOps Skills / 优秀的 DevOps AI 技能集

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

> **Stop firefighting. 12 AI skills to automate your infrastructure.** / **停止救火模式。12 个 AI 技能帮你自动化基础设施。**

A curated collection of production-ready AI skills for DevOps and infrastructure automation. Each skill file is a self-contained knowledge module that an AI agent can load to help you build, deploy, monitor, and operate production systems.

一套面向 DevOps 和基础设施自动化的生产级 AI 技能集。每个技能文件都是一个独立的知识模块，AI 代理可以直接加载使用，帮助你构建、部署、监控和运维生产系统。

---

## Table of Contents / 目录

- [Skills Overview / 技能总览](#skills-overview--技能总览)
- [Quick Start / 快速开始](#quick-start--快速开始)
- [CI/CD](#cicd)
- [容器化 / Containerization](#容器化--containerization)
- [云服务 / Cloud](#云服务--cloud)
- [监控告警 / Monitoring](#监控告警--monitoring)
- [Contributing / 贡献指南](#contributing--贡献指南)
- [See Also / 相关项目](#see-also--相关项目)

---

## Skills Overview / 技能总览

| # | Category / 分类 | Skill / 技能 | Description / 描述 |
|---|---|---|---|
| 1 | CI/CD | [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md) | Matrix builds, OIDC, reusable workflows, caching / 矩阵构建、OIDC、可复用工作流、缓存 |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Runners, DAG pipelines, environments, security scanning / 运行器、DAG 流水线、环境管理、安全扫描 |
| 3 | CI/CD | [Deployment Strategies](skills/CI-CD/deployment-strategies.md) | Blue-green, canary, rolling, Argo Rollouts / 蓝绿、金丝雀、滚动部署、Argo Rollouts |
| 4 | 容器化 | [Docker Production](skills/容器化/docker-production.md) | Multi-stage builds, security hardening, image optimization / 多阶段构建、安全加固、镜像优化 |
| 5 | 容器化 | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, networking, troubleshooting / 部署、RBAC、HPA、网络、故障排查 |
| 6 | 容器化 | [Helm Charts](skills/容器化/helm-charts.md) | Chart development, templating, testing, repositories / Chart 开发、模板化、测试、仓库管理 |
| 7 | 云服务 | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, cost optimization / IAM、VPC、ECS、RDS、S3、成本优化 |
| 8 | 云服务 | [Terraform IaC](skills/云服务/terraform-iac.md) | Modules, state management, workspaces, CI/CD integration / 模块、状态管理、工作区、CI/CD 集成 |
| 9 | 云服务 | [Serverless Patterns](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge / Lambda、API Gateway、Step Functions、EventBridge |
| 10 | 监控告警 | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporters, PromQL, dashboards, SLO-based alerts / Exporter、PromQL、仪表盘、基于 SLO 的告警 |
| 11 | 监控告警 | [Logging Stack](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, retention / ELK、Loki、Fluentd、Vector、LogQL、日志保留 |
| 12 | 监控告警 | [Incident Response](skills/监控告警/incident-response.md) | On-call, runbooks, post-mortems, escalation / 值班、Runbook、事后复盘、升级策略 |

---

## Quick Start / 快速开始

### For AI Agent Users / AI 代理用户

1. **Choose a skill** from the table above based on your task / 根据任务从上表选择技能
2. **Load the skill file** into your AI agent's context / 将技能文件加载到 AI 代理的上下文中
3. **Apply the templates** — each skill contains production-ready code / 应用模板——每个技能包含生产级代码

### For Developers / 开发者

```bash
# Clone the repository / 克隆仓库
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills / 浏览技能
ls skills/*/

# Use in your project / 在项目中使用
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
cp skills/容器化/helm-charts.md ./charts/
```

### Skill File Structure / 技能文件结构

Each skill follows a consistent format / 每个技能遵循统一格式：

```
┌─────────────────────────────────┐
│  When to Use / 何时使用          │  ← Clear trigger conditions
│  Architecture / 架构             │  ← ASCII diagrams
│  Code Templates / 代码模板       │  ← YAML/HCL/Dockerfile
│  Best Practices / 最佳实践       │  ← Battle-tested recommendations
│  Pitfalls / 常见陷阱             │  ← Common mistakes to avoid
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

## 容器化 / Containerization

### [Docker Production](skills/容器化/docker-production.md)
Production Docker patterns: multi-stage builds (45MB images), non-root containers, security scanning with Trivy, multi-architecture builds, and Docker Compose for production.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
K8s production patterns: deployments with topology spread, RBAC least-privilege, HPA with custom metrics, NetworkPolicies, and comprehensive troubleshooting commands.

### [Helm Charts](skills/容器化/helm-charts.md)
Helm best practices: modular chart structure, template helpers, multi-environment values, `--atomic` deployments, chart testing, and OCI registry publishing.

---

## 云服务 / Cloud

### [AWS Essentials](skills/云服务/aws-essentials.md)
AWS production architecture: VPC with multi-AZ, IAM least-privilege with OIDC, ECS Fargate services, Aurora Serverless, S3 security, and cost optimization with budgets.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform at scale: reusable modules, remote state with locking, environment separation, CI/CD with plan/apply, Terratest, and state migration strategies.

### [Serverless Patterns](skills/云服务/serverless-patterns.md)
Event-driven serverless: Lambda with API Gateway, Step Functions workflows, EventBridge rules, DynamoDB single-table design, cold start optimization, and SAM templates.

---

## 监控告警 / Monitoring

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Full observability stack: Prometheus configuration, recording rules, SLO-based alerting, Grafana dashboards with variables, Alertmanager routing, and Thanos for long-term storage.

### [Logging Stack](skills/监控告警/logging-stack.md)
Centralized logging: Grafana Loki deployment, Fluent Bit/Vector log shipping, LogQL queries, Elasticsearch ILM policies, structured logging, and log-based alerts.

### [Incident Response](skills/监控告警/incident-response.md)
Incident management: PagerDuty escalation policies, runbook templates, post-mortem format, severity definitions, communication templates, and SLO error budget tracking.

---

## Contributing / 贡献指南

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解贡献指南。

**How to contribute / 如何贡献:**

1. Fork the repository / Fork 仓库
2. Create a feature branch / 创建功能分支
3. Add or improve a skill file / 添加或改进技能文件
4. Submit a Pull Request / 提交 Pull Request

**Skill requirements / 技能要求:**
- 150+ lines per file / 每个文件 150 行以上
- Must include all 5 sections / 必须包含全部 5 个章节
- Production-ready code templates / 提供生产级代码模板
- Bilingual section headers (EN/CN) / 中英双语章节标题

---

## See Also / 相关项目

Other awesome collections you might find useful / 你可能感兴趣的其他优秀项目：

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Self-hosted software / 自托管软件
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetes resources / Kubernetes 资源
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Docker ecosystem / Docker 生态
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraform modules and tools / Terraform 模块与工具
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheus ecosystem / Prometheus 生态
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWS resources / AWS 资源
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Site Reliability Engineering / 站点可靠性工程
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm charts and tools / Helm Chart 与工具
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CD pipeline tools / CI/CD 流水线工具
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafana dashboards and plugins / Grafana 仪表盘与插件
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Incident management resources / 故障管理资源
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Serverless frameworks and tools / Serverless 框架与工具

---

## License / 许可证

[MIT License](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Made with care for the DevOps community / 为 DevOps 社区用心打造<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Star this repo</a> if you find it useful / 觉得有用请点 Star
</p>
