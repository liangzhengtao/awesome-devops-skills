[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# Awesome DevOps Skills

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

> **Hören Sie auf, Brände zu löschen. 12 KI-Skills zur Automatisierung Ihrer Infrastruktur.**

Eine kuratierte Sammlung von produktionsreifen KI-Skills für DevOps und Infrastrukturautomatisierung. Jede Skill-Datei ist ein eigenständiges Wissensmodul, das ein KI-Agent laden kann, um Ihnen beim Aufbau, Deployment, Monitoring und Betrieb von Produktionssystemen zu helfen.

---

## Inhaltsverzeichnis

- [Skills-Übersicht](#skills-overview)
- [Schnellstart](#quick-start)
- [CI/CD](#cicd)
- [Containerisierung](#containerization)
- [Cloud](#cloud)
- [Monitoring](#monitoring)
- [Mitwirken](#contributing)
- [Siehe auch](#see-also)

---

## Skills-Übersicht

| # | Kategorie | Skill | Beschreibung |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md) | Matrix-Builds, OIDC, wiederverwendbare Workflows, Caching |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Runner, DAG-Pipelines, Umgebungen, Sicherheitsscans |
| 3 | CI/CD | [Deployment-Strategien](skills/CI-CD/deployment-strategies.md) | Blue-Green, Canary, Rolling, Argo Rollouts |
| 4 | Containerisierung | [Docker in Produktion](skills/容器化/docker-production.md) | Multi-Stage-Builds, Sicherheitshärtung, Image-Optimierung |
| 5 | Containerisierung | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, Netzwerk, Troubleshooting |
| 6 | Containerisierung | [Helm Charts](skills/容器化/helm-charts.md) | Chart-Entwicklung, Templates, Tests, Repositories |
| 7 | Cloud | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, Kostenoptimierung |
| 8 | Cloud | [Terraform IaC](skills/云服务/terraform-iac.md) | Module, State-Management, Workspaces, CI/CD-Integration |
| 9 | Cloud | [Serverless-Pattern](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | Monitoring | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporter, PromQL, Dashboards, SLO-basierte Alerts |
| 11 | Monitoring | [Logging-Stack](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, Aufbewahrung |
| 12 | Monitoring | [Incident Response](skills/监控告警/incident-response.md) | On-Call, Runbooks, Post-Mortems, Eskalation |

---

## Schnellstart

### Für KI-Agent-Nutzer

1. **Wählen Sie einen Skill** aus der Tabelle oben basierend auf Ihrer Aufgabe
2. **Laden Sie die Skill-Datei** in den Kontext Ihres KI-Agenten
3. **Wenden Sie die Templates an** — jeder Skill enthält Produktionscode

### Für Entwickler

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### Skill-Dateistruktur

Jeder Skill folgt einem einheitlichen Format:

```
┌─────────────────────────────────┐
│  Wann verwenden                 │  ← Klare Auslösebedingungen
│  Architektur                    │  ← ASCII-Diagramme
│  Code-Templates                 │  ← YAML/HCL/Dockerfile
│  Bewährte Pattern               │  ← Erprobte Empfehlungen
│  Fallstricke                    │  ← Häufige Fehler vermeiden
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md)
Fortgeschrittene GitHub Actions Pattern: Matrix-Builds, OIDC-Authentifizierung, wiederverwendbare Workflows, Caching-Strategien, Self-Hosted Runner und Semantic-Release-Automatisierung.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
Produktionsreife GitLab-Pipelines: DAG-Optimierung, Auto-Scaling-Runner, Review Apps, Security-Scanning-Templates und Multi-Environment-Deployments.

### [Deployment-Strategien](skills/CI-CD/deployment-strategies.md)
Zero-Downtime-Deployment-Pattern: Blue-Green mit Argo Rollouts, Canary mit Traffic-Splitting und automatisierter Analyse, Rolling Updates mit geeigneten Health Checks.

---

## Containerisierung

### [Docker in Produktion](skills/容器化/docker-production.md)
Produktionsreife Docker-Pattern: Multi-Stage-Builds (45MB-Images), Non-Root-Container, Sicherheitsscans mit Trivy, Multi-Architecture-Builds und Docker Compose für Produktion.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
K8s-Produktionspattern: Deployments mit Topology Spread, RBAC mit Least Privilege, HPA mit Custom Metrics, NetworkPolicies und vollständige Troubleshooting-Befehle.

### [Helm Charts](skills/容器化/helm-charts.md)
Bewährte Helm-Pattern: Modulare Chart-Struktur, Template-Helpers, Multi-Environment-Werte, `--atomic`-Deployments, Chart-Testing und OCI-Registry-Veröffentlichung.

---

## Cloud

### [AWS Essentials](skills/云服务/aws-essentials.md)
AWS-Produktionsarchitektur: VPC mit Multi-AZ, IAM mit Least Privilege und OIDC, ECS Fargate-Services, Aurora Serverless, S3-Sicherheit und Kostenoptimierung mit Budgets.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform im großen Maßstab: Wiederverwendbare Module, Remote State mit Locking, Umgebungstrennung, CI/CD mit plan/apply, Terratest und State-Migrationsstrategien.

### [Serverless-Pattern](skills/云服务/serverless-patterns.md)
Event-getriebenes Serverless: Lambda mit API Gateway, Step Functions-Workflows, EventBridge-Regeln, DynamoDB Single-Table-Design, Cold-Start-Optimierung und SAM-Templates.

---

## Monitoring

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Vollständiger Observability-Stack: Prometheus-Konfiguration, Recording Rules, SLO-basiertes Alerting, Grafana-Dashboards mit Variablen, Alertmanager-Routing und Thanos für Langzeitspeicherung.

### [Logging-Stack](skills/监控告警/logging-stack.md)
Zentralisiertes Logging: Grafana Loki-Deployment, Fluent Bit/Vector Log-Shipping, LogQL-Abfragen, Elasticsearch ILM-Policies, strukturiertes Logging und log-basierte Alerts.

### [Incident Response](skills/监控告警/incident-response.md)
Incident Management: PagerDuty-Eskalationsrichtlinien, Runbook-Templates, Post-Mortem-Format, Schweregrad-Definitionen, Kommunikations-Templates und SLO-Error-Budget-Tracking.

---

## Mitwirken

Beiträge sind willkommen! Siehe [CONTRIBUTING.md](CONTRIBUTING.md) für Richtlinien.

**So tragen Sie bei:**

1. Forken Sie das Repository
2. Erstellen Sie einen Feature-Branch
3. Fügen Sie eine Skill-Datei hinzu oder verbessern Sie sie
4. Reichen Sie einen Pull Request ein

**Skill-Anforderungen:**

- 150+ Zeilen pro Datei
- Alle 5 Abschnitte müssen enthalten sein
- Produktionsreife Code-Templates

---

## Siehe auch

Weitere großartige Sammlungen, die nützlich sein könnten:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Self-Hosted Software
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetes-Ressourcen
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Docker-Ökosystem
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraform-Module und -Tools
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheus-Ökosystem
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWS-Ressourcen
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Site Reliability Engineering
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm-Charts und -Tools
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CD-Pipeline-Tools
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafana-Dashboards und -Plugins
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Incident-Management-Ressourcen
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Serverless-Frameworks und -Tools

---

## Lizenz

[MIT-Lizenz](LICENSE) — Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Mit Sorgfalt für die DevOps-Community erstellt<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Vergeben Sie einen Stern</a>, wenn Sie dieses Repository nützlich finden
</p>
