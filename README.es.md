[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md)

<div align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# Awesome DevOps Skills

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

> **Deja de apagar incendios. 12 habilidades IA para automatizar tu infraestructura.**

Una colección curada de habilidades IA de producción para DevOps y automatización de infraestructura. Cada archivo de habilidad es un módulo de conocimiento independiente que un agente IA puede cargar para ayudarte a construir, desplegar, monitorizar y operar sistemas en producción.

---

## Tabla de contenidos

- [Resumen de habilidades](#resumen-de-habilidades)
- [Inicio rápido](#inicio-rápido)
- [CI/CD](#cicd)
- [Contenedores](#contenedores)
- [Cloud](#cloud)
- [Monitorización](#monitorización)
- [Contribuir](#contribuir)
- [Ver también](#ver-también)

---

## Resumen de habilidades

| # | Categoría | Habilidad | Descripción |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Avanzado](skills/CI-CD/github-actions-advanced.md) | Builds matriciales, OIDC, workflows reutilizables, caché |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Runners, pipelines DAG, entornos, análisis de seguridad |
| 3 | CI/CD | [Estrategias de despliegue](skills/CI-CD/deployment-strategies.md) | Blue-green, canary, rolling, Argo Rollouts |
| 4 | Contenedores | [Docker en producción](skills/容器化/docker-production.md) | Multi-stage, endurecimiento de seguridad, optimización de imágenes |
| 5 | Contenedores | [Kubernetes Esencial](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, redes, resolución de problemas |
| 6 | Contenedores | [Helm Charts](skills/容器化/helm-charts.md) | Desarrollo de charts, plantillas, testing, repositorios |
| 7 | Cloud | [AWS Esencial](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, optimización de costes |
| 8 | Cloud | [Terraform IaC](skills/云服务/terraform-iac.md) | Módulos, gestión de estado, workspaces, integración CI/CD |
| 9 | Cloud | [Patrones Serverless](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | Monitorización | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exportadores, PromQL, dashboards, alertas basadas en SLO |
| 11 | Monitorización | [Stack de logging](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, retención |
| 12 | Monitorización | [Respuesta a incidentes](skills/监控告警/incident-response.md) | Guardia, runbooks, post-mortems, escalación |

---

## Inicio rápido

### Para usuarios de agentes IA

1. **Elige una habilidad** de la tabla superior según tu tarea
2. **Carga el archivo de habilidad** en el contexto de tu agente IA
3. **Aplica las plantillas** — cada habilidad contiene código de producción

### Para desarrolladores

```bash
# Clonar el repositorio
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Explorar habilidades
ls skills/*/

# Usar en tu proyecto
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### Estructura del archivo de habilidad

Cada habilidad sigue un formato coherente:

```
┌─────────────────────────────────┐
│  Cuándo usarla                  │  ← Condiciones de activación claras
│  Arquitectura                   │  ← Diagramas ASCII
│  Plantillas de código           │  ← YAML/HCL/Dockerfile
│  Patrones probados              │  ← Recomendaciones contrastadas
│  Errores comunes                │  ← Errores frecuentes a evitar
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Avanzado](skills/CI-CD/github-actions-advanced.md)
Patrones avanzados para GitHub Actions: builds matriciales, autenticación OIDC, workflows reutilizables, estrategias de caché, runners autohospedados y automatización de publicación semántica.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
Pipelines GitLab de producción: optimización DAG, runners con autoescalado, Review Apps, plantillas de análisis de seguridad y despliegues multi-entorno.

### [Estrategias de despliegue](skills/CI-CD/deployment-strategies.md)
Patrones de despliegue sin tiempo de inactividad: blue-green con Argo Rollouts, canary con división de tráfico y análisis automático, rolling updates con health checks.

---

## Contenedores

### [Docker en producción](skills/容器化/docker-production.md)
Patrones Docker de producción: multi-stage (imágenes de 45MB), contenedores no-root, análisis de seguridad con Trivy, builds multi-arquitectura y Docker Compose para producción.

### [Kubernetes Esencial](skills/容器化/kubernetes-essentials.md)
Patrones K8s de producción: despliegues con distribución topológica, RBAC de mínimo privilegio, HPA con métricas personalizadas, NetworkPolicy y comandos completos de troubleshooting.

### [Helm Charts](skills/容器化/helm-charts.md)
Patrones Helm probados: estructura modular de charts, helpers de template, values multi-entorno, despliegues `--atomic`, testing de charts y publicación en registro OCI.

---

## Cloud

### [AWS Esencial](skills/云服务/aws-essentials.md)
Arquitectura AWS de producción: VPC multi-AZ, IAM OIDC de mínimo privilegio, servicios ECS Fargate, Aurora Serverless, seguridad S3 y optimización de costes con presupuestos.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform a gran escala: módulos reutilizables, estado remoto con bloqueo, separación de entornos, CI/CD con plan/apply, Terratest y estrategias de migración de estado.

### [Patrones Serverless](skills/云服务/serverless-patterns.md)
Serverless impulsado por eventos: Lambda con API Gateway, workflows Step Functions, reglas EventBridge, DynamoDB single-table, optimización de cold start y plantillas SAM.

---

## Monitorización

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Stack completo de observabilidad: configuración Prometheus, recording rules, alertas basadas en SLO, dashboards Grafana con variables, enrutamiento Alertmanager y Thanos para almacenamiento a largo plazo.

### [Stack de logging](skills/监控告警/logging-stack.md)
Logging centralizado: despliegue Grafana Loki, envío de logs Fluent Bit/Vector, consultas LogQL, políticas ILM Elasticsearch, logging estructurado y alertas basadas en logs.

### [Respuesta a incidentes](skills/监控告警/incident-response.md)
Gestión de incidentes: políticas de escalación PagerDuty, plantillas de runbook, formato post-mortem, definiciones de severidad, plantillas de comunicación y seguimiento de presupuesto de error SLO.

---

## Contribuir

¡Las contribuciones son bienvenidas! Consulta [CONTRIBUTING.md](CONTRIBUTING.md) para las directrices.

**Cómo contribuir:**

1. Haz fork del repositorio
2. Crea una rama de funcionalidad
3. Añade o mejora un archivo de habilidad
4. Envía un Pull Request

**Requisitos de habilidades:**
- 150+ líneas por archivo
- Debe incluir las 5 secciones
- Plantillas de código de producción

---

## Ver también

Otras colecciones awesome que podrían resultarte útiles:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Software autoalojado
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Recursos Kubernetes
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Ecosistema Docker
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Módulos y herramientas Terraform
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Ecosistema Prometheus
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — Recursos AWS
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Ingeniería de Fiabilidad del Sitio
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Charts y herramientas Helm
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — Herramientas de pipeline CI/CD
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Dashboards y plugins Grafana
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Recursos de gestión de incidentes
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Frameworks y herramientas Serverless

---

## Licencia

[MIT License](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Hecho con dedicación para la comunidad DevOps<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Dale una estrella</a> si te resulta útil
</p>
