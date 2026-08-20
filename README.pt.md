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

> **Pare de apagar incêndios. 12 habilidades de IA para automatizar sua infraestrutura.**

Uma coleção curada de habilidades de IA para produção em DevOps e automação de infraestrutura. Cada arquivo de habilidade é um módulo de conhecimento autônomo que um agente de IA pode carregar para ajudá-lo a construir, implantar, monitorar e operar sistemas de produção.

---

## Sumário

- [Visão Geral das Habilidades](#skills-overview)
- [Início Rápido](#quick-start)
- [CI/CD](#cicd)
- [Containerização](#containerization)
- [Cloud](#cloud)
- [Monitoramento](#monitoring)
- [Contribuindo](#contributing)
- [Veja Também](#see-also)

---

## Visão Geral das Habilidades

| # | Categoria | Habilidade | Descrição |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Avançado](skills/CI-CD/github-actions-advanced.md) | Builds em matriz, OIDC, workflows reutilizáveis, cache |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Runners, pipelines DAG, ambientes, análise de segurança |
| 3 | CI/CD | [Estratégias de Implantação](skills/CI-CD/deployment-strategies.md) | Blue-green, canary, rolling, Argo Rollouts |
| 4 | Containerização | [Docker em Produção](skills/容器化/docker-production.md) | Builds multi-stage, hardening de segurança, otimização de imagens |
| 5 | Containerização | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, networking, troubleshooting |
| 6 | Containerização | [Helm Charts](skills/容器化/helm-charts.md) | Desenvolvimento de charts, templates, testes, repositórios |
| 7 | Cloud | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, otimização de custos |
| 8 | Cloud | [Terraform IaC](skills/云服务/terraform-iac.md) | Módulos, gerenciamento de estado, workspaces, integração CI/CD |
| 9 | Cloud | [Padrões Serverless](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | Monitoramento | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporters, PromQL, dashboards, alertas baseados em SLO |
| 11 | Monitoramento | [Stack de Logging](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, retenção |
| 12 | Monitoramento | [Resposta a Incidentes](skills/监控告警/incident-response.md) | On-call, runbooks, post-mortems, escalação |

---

## Início Rápido

### Para Usuários de Agentes de IA

1. **Escolha uma habilidade** da tabela acima com base na sua tarefa
2. **Carregue o arquivo de habilidade** no contexto do seu agente de IA
3. **Aplique os templates** — cada habilidade contém código de produção

### Para Desenvolvedores

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### Estrutura do Arquivo de Habilidade

Cada habilidade segue um formato consistente:

```
┌─────────────────────────────────┐
│  Quando Usar                    │  ← Condições de gatilho claras
│  Arquitetura                    │  ← Diagramas ASCII
│  Templates de Código            │  ← YAML/HCL/Dockerfile
│  Padrões Comprovados            │  ← Recomendações testadas
│  Armadilhas                     │  ← Erros comuns a evitar
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Avançado](skills/CI-CD/github-actions-advanced.md)
Padrões avançados para GitHub Actions: builds em matriz, autenticação OIDC, workflows reutilizáveis, estratégias de cache, runners auto-hospedados e automação de release semântico.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
Pipelines GitLab de produção: otimização DAG, runners com auto-scaling, review apps, templates de análise de segurança e implantações multi-ambiente.

### [Estratégias de Implantação](skills/CI-CD/deployment-strategies.md)
Padrões de implantação sem downtime: blue-green com Argo Rollouts, canary com divisão de tráfego e análise automatizada, rolling updates com health checks adequados.

---

## Containerização

### [Docker em Produção](skills/容器化/docker-production.md)
Padrões Docker de produção: builds multi-stage (imagens de 45MB), containers non-root, análise de segurança com Trivy, builds multi-arquitetura e Docker Compose para produção.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
Padrões K8s de produção: deployments com topology spread, RBAC de menor privilégio, HPA com métricas customizadas, NetworkPolicies e comandos completos de troubleshooting.

### [Helm Charts](skills/容器化/helm-charts.md)
Padrões Helm comprovados: estrutura modular de charts, helpers de template, valores multi-ambiente, implantações `--atomic`, teste de charts e publicação em registry OCI.

---

## Cloud

### [AWS Essentials](skills/云服务/aws-essentials.md)
Arquitetura AWS de produção: VPC com multi-AZ, IAM de menor privilégio com OIDC, serviços ECS Fargate, Aurora Serverless, segurança S3 e otimização de custos com orçamentos.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform em escala: módulos reutilizáveis, estado remoto com lock, separação de ambientes, CI/CD com plan/apply, Terratest e estratégias de migração de estado.

### [Padrões Serverless](skills/云服务/serverless-patterns.md)
Serverless orientado a eventos: Lambda com API Gateway, workflows Step Functions, regras EventBridge, design de tabela única DynamoDB, otimização de cold start e templates SAM.

---

## Monitoramento

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Stack completo de observabilidade: configuração Prometheus, regras de gravação, alertas baseados em SLO, dashboards Grafana com variáveis, roteamento Alertmanager e Thanos para armazenamento de longo prazo.

### [Stack de Logging](skills/监控告警/logging-stack.md)
Logging centralizado: implantação Grafana Loki, envio de logs Fluent Bit/Vector, consultas LogQL, políticas ILM do Elasticsearch, logging estruturado e alertas baseados em logs.

### [Resposta a Incidentes](skills/监控告警/incident-response.md)
Gerenciamento de incidentes: políticas de escalação PagerDuty, templates de runbook, formato post-mortem, definições de severidade, templates de comunicação e rastreamento de orçamento de erro SLO.

---

## Contribuindo

Contribuições são bem-vindas! Veja [CONTRIBUTING.md](CONTRIBUTING.md) para diretrizes.

**Como contribuir:**

1. Faça um fork do repositório
2. Crie uma branch de feature
3. Adicione ou melhore um arquivo de habilidade
4. Envie um Pull Request

**Requisitos da habilidade:**

- 150+ linhas por arquivo
- Deve incluir todas as 5 seções
- Templates de código de produção

---

## Veja Também

Outras coleções incríveis que você pode achar úteis:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Software auto-hospedado
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Recursos Kubernetes
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Ecossistema Docker
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Módulos e ferramentas Terraform
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Ecossistema Prometheus
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — Recursos AWS
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Engenharia de Confiabilidade do Site
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Charts e ferramentas Helm
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — Ferramentas de pipeline CI/CD
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Dashboards e plugins Grafana
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Recursos de gerenciamento de incidentes
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Frameworks e ferramentas serverless

---

## Licença

[Licença MIT](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Feito com carinho para a comunidade DevOps<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Dê uma estrela neste repo</a> se achou útil
</p>
