# Awesome DevOps Skills / 优秀的 DevOps AI 技能集

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

<p align="center">
  <a href="#english">English</a> | <a href="#中文">中文</a>
</p>

---

<a name="english"></a>

> **Stop firefighting. 12 AI skills to automate your infrastructure.**

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
│  Proven Patterns / 最佳实践       │  ← Battle-tested recommendations
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
K8s production patterns: deployments with topology spread, RBAC least-privilege, HPA with custom metrics, NetworkPolicies, and complete troubleshooting commands.

### [Helm Charts](skills/容器化/helm-charts.md)
Helm proven patterns: modular chart structure, template helpers, multi-environment values, `--atomic` deployments, chart testing, and OCI registry publishing.

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

---

<a name="中文"></a>

# 优秀的 DevOps AI 技能集

> **停止救火模式。12 个 AI 技能帮你自动化基础设施。**

一套面向 DevOps 和基础设施自动化的生产级 AI 技能集。每个技能文件都是一个独立的知识模块，AI 代理可以直接加载使用，帮助你构建、部署、监控和运维生产系统。

---

## 目录

- [技能总览](#技能总览)
- [快速开始](#快速开始)
- [CI/CD](#cicd-1)
- [容器化](#容器化)
- [云服务](#云服务)
- [监控告警](#监控告警)
- [贡献指南](#贡献指南)
- [相关项目](#相关项目)

---

## 技能总览

| # | 分类 | 技能 | 描述 |
|---|------|------|------|
| 1 | CI/CD | [GitHub Actions 进阶](skills/CI-CD/github-actions-advanced.md) | 矩阵构建、OIDC、可复用工作流、缓存 |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | 运行器、DAG 流水线、环境管理、安全扫描 |
| 3 | CI/CD | [部署策略](skills/CI-CD/deployment-strategies.md) | 蓝绿、金丝雀、滚动部署、Argo Rollouts |
| 4 | 容器化 | [Docker 生产实践](skills/容器化/docker-production.md) | 多阶段构建、安全加固、镜像优化 |
| 5 | 容器化 | [Kubernetes 核心](skills/容器化/kubernetes-essentials.md) | 部署、RBAC、HPA、网络、故障排查 |
| 6 | 容器化 | [Helm Charts](skills/容器化/helm-charts.md) | Chart 开发、模板化、测试、仓库管理 |
| 7 | 云服务 | [AWS 核心](skills/云服务/aws-essentials.md) | IAM、VPC、ECS、RDS、S3、成本优化 |
| 8 | 云服务 | [Terraform IaC](skills/云服务/terraform-iac.md) | 模块、状态管理、工作区、CI/CD 集成 |
| 9 | 云服务 | [Serverless 模式](skills/云服务/serverless-patterns.md) | Lambda、API Gateway、Step Functions、EventBridge |
| 10 | 监控告警 | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporter、PromQL、仪表盘、基于 SLO 的告警 |
| 11 | 监控告警 | [日志系统](skills/监控告警/logging-stack.md) | ELK、Loki、Fluentd、Vector、LogQL、日志保留 |
| 12 | 监控告警 | [故障响应](skills/监控告警/incident-response.md) | 值班、Runbook、事后复盘、升级策略 |

---

## 快速开始

### AI 代理用户

1. 根据任务从上表**选择技能**
2. 将技能文件**加载到 AI 代理的上下文**中
3. **应用模板**——每个技能包含生产级代码

### 开发者

```bash
# 克隆仓库
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# 浏览技能
ls skills/*/

# 在项目中使用
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
cp skills/容器化/helm-charts.md ./charts/
```

### 技能文件结构

每个技能遵循统一格式：

```
┌─────────────────────────────────┐
│  何时使用                        │  ← 明确的触发条件
│  架构                            │  ← ASCII 图表
│  代码模板                        │  ← YAML/HCL/Dockerfile
│  最佳实践                        │  ← 经过实战验证的建议
│  常见陷阱                        │  ← 需要避免的常见错误
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions 进阶](skills/CI-CD/github-actions-advanced.md)
GitHub Actions 高级模式：矩阵构建、OIDC 认证、可复用工作流、缓存策略、自托管运行器和语义化发布自动化。

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
生产级 GitLab 流水线：DAG 优化、自动扩缩运行器、Review Apps、安全扫描模板和多环境部署。

### [部署策略](skills/CI-CD/deployment-strategies.md)
零停机部署模式：基于 Argo Rollouts 的蓝绿部署、带流量分割和自动分析的金丝雀发布、带健康检查的滚动更新。

---

## 容器化

### [Docker 生产实践](skills/容器化/docker-production.md)
生产级 Docker 模式：多阶段构建（45MB 镜像）、非 root 容器、Trivy 安全扫描、多架构构建和生产级 Docker Compose。

### [Kubernetes 核心](skills/容器化/kubernetes-essentials.md)
K8s 生产模式：拓扑分布部署、RBAC 最小权限、自定义指标 HPA、NetworkPolicy 和全面的故障排查命令。

### [Helm Charts](skills/容器化/helm-charts.md)
Helm 最佳实践：模块化 Chart 结构、模板助手、多环境 values、`--atomic` 部署、Chart 测试和 OCI 仓库发布。

---

## 云服务

### [AWS 核心](skills/云服务/aws-essentials.md)
AWS 生产架构：多可用区 VPC、OIDC IAM 最小权限、ECS Fargate 服务、Aurora Serverless、S3 安全和预算成本优化。

### [Terraform IaC](skills/云服务/terraform-iac.md)
大规模 Terraform：可复用模块、带锁的远程状态、环境隔离、CI/CD plan/apply、Terratest 和状态迁移策略。

### [Serverless 模式](skills/云服务/serverless-patterns.md)
事件驱动 Serverless：Lambda + API Gateway、Step Functions 工作流、EventBridge 规则、DynamoDB 单表设计、冷启动优化和 SAM 模板。

---

## 监控告警

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
完整可观测性栈：Prometheus 配置、Recording Rules、基于 SLO 的告警、带变量的 Grafana 仪表盘、Alertmanager 路由和 Thanos 长期存储。

### [日志系统](skills/监控告警/logging-stack.md)
集中式日志：Grafana Loki 部署、Fluent Bit/Vector 日志传输、LogQL 查询、Elasticsearch ILM 策略、结构化日志和基于日志的告警。

### [故障响应](skills/监控告警/incident-response.md)
故障管理：PagerDuty 升级策略、Runbook 模板、事后复盘格式、严重性定义、沟通模板和 SLO 错误预算追踪。

---

## 贡献指南

欢迎贡献！请查看 [CONTRIBUTING.md](CONTRIBUTING.md) 了解贡献指南。

**如何贡献：**

1. Fork 仓库
2. 创建功能分支
3. 添加或改进技能文件
4. 提交 Pull Request

**技能要求：**
- 每个文件 150 行以上
- 必须包含全部 5 个章节
- 提供生产级代码模板
- 中英双语章节标题

---

## 相关项目

你可能感兴趣的其他优秀项目：

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — 自托管软件
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetes 资源
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Docker 生态
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraform 模块与工具
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheus 生态
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWS 资源
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — 站点可靠性工程
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm Chart 与工具
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CD 流水线工具
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafana 仪表盘与插件
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — 故障管理资源
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Serverless 框架与工具

---

## 许可证

[MIT 许可证](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  为 DevOps 社区用心打造<br>
  觉得有用请 <a href="https://github.com/liangzhengtao/awesome-devops-skills">点 Star</a>
</p>
