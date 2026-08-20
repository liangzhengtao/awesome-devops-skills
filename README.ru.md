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

> **Перестаньте тушить пожары. 12 ИИ-навыков для автоматизации вашей инфраструктуры.**

Тщательно подобранная коллекция продуктовых ИИ-навыков для DevOps и автоматизации инфраструктуры. Каждый файл навыка — это автономный модуль знаний, который ИИ-агент может загрузить, чтобы помочь вам создавать, развёртывать, мониторить и эксплуатировать продакшн-системы.

---

## Содержание

- [Обзор навыков](#skills-overview)
- [Быстрый старт](#quick-start)
- [CI/CD](#cicd)
- [Контейнеризация](#containerization)
- [Облако](#cloud)
- [Мониторинг](#monitoring)
- [Участие](#contributing)
- [Также посмотрите](#see-also)

---

## Обзор навыков

| # | Категория | Навык | Описание |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md) | Матричные сборки, OIDC, переиспользуемые воркфлоу, кеширование |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | Раннеры, DAG-пайплайны, окружения, сканирование безопасности |
| 3 | CI/CD | [Стратегии деплоя](skills/CI-CD/deployment-strategies.md) | Blue-green, canary, rolling, Argo Rollouts |
| 4 | Контейнеризация | [Docker в продакшне](skills/容器化/docker-production.md) | Многостадийные сборки, укрепление безопасности, оптимизация образов |
| 5 | Контейнеризация | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | Deployments, RBAC, HPA, сети, устранение неполадок |
| 6 | Контейнеризация | [Helm Charts](skills/容器化/helm-charts.md) | Разработка чартов, шаблоны, тестирование, репозитории |
| 7 | Облако | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, оптимизация затрат |
| 8 | Облако | [Terraform IaC](skills/云服务/terraform-iac.md) | Модули, управление состоянием, воркспейсы, интеграция CI/CD |
| 9 | Облако | [Serverless-паттерны](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | Мониторинг | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Экспортёры, PromQL, дашборды, алерты на основе SLO |
| 11 | Мониторинг | [Стек логирования](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, хранение |
| 12 | Мониторинг | [Реагирование на инциденты](skills/监控告警/incident-response.md) | Он-колл, ранбуки, постмортемы, эскалация |

---

## Быстрый старт

### Для пользователей ИИ-агентов

1. **Выберите навык** из таблицы выше на основе вашей задачи
2. **Загрузите файл навыка** в контекст вашего ИИ-агента
3. **Примените шаблоны** — каждый навык содержит продакшн-код

### Для разработчиков

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### Структура файла навыка

Каждый навык следует единообразному формату:

```
┌─────────────────────────────────┐
│  Когда использовать             │  ← Чёткие условия срабатывания
│  Архитектура                    │  ← ASCII-диаграммы
│  Шаблоны кода                   │  ← YAML/HCL/Dockerfile
│  Проверенные паттерны           │  ← Рекомендации из практики
│  Подводные камни                │  ← Частые ошибки
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md)
Продвинутые паттерны GitHub Actions: матричные сборки, аутентификация OIDC, переиспользуемые воркфлоу, стратегии кеширования, самохостируемые раннеры и автоматизация семантического релиза.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
Продакшн-пайплайны GitLab: DAG-оптимизация, раннеры с автоскейлингом, ревью-приложения, шаблоны сканирования безопасности и мульти-окружные деплои.

### [Стратегии деплоя](skills/CI-CD/deployment-strategies.md)
Паттерны деплоя без даунтайма: blue-green с Argo Rollouts, canary с распределением трафика и автоматическим анализом, rolling updates с проверками состояния здоровья.

---

## Контейнеризация

### [Docker в продакшне](skills/容器化/docker-production.md)
Продакшн-паттерны Docker: многостадийные сборки (образы 45MB), non-root контейнеры, сканирование безопасности с Trivy, мульти-архитектурные сборки и Docker Compose для продакшена.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
Продакшн-паттерны K8s: деплои с topology spread, RBAC с минимальными привилегиями, HPA с пользовательскими метриками, NetworkPolicies и полные команды устранения неполадок.

### [Helm Charts](skills/容器化/helm-charts.md)
Проверенные паттерны Helm: модульная структура чартов, хелперы шаблонов, значения для нескольких окружений, `--atomic` деплои, тестирование чартов и публикация в OCI-реестр.

---

## Облако

### [AWS Essentials](skills/云服务/aws-essentials.md)
Продакшн-архитектура AWS: VPC с мульти-AZ, IAM с минимальными привилегиями и OIDC, сервисы ECS Fargate, Aurora Serverless, безопасность S3 и оптимизация затрат с бюджетами.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform в масштабе: переиспользуемые модули, удалённое состояние с блокировкой, разделение окружений, CI/CD с plan/apply, Terratest и стратегии миграции состояния.

### [Serverless-паттерны](skills/云服务/serverless-patterns.md)
Ивент-драйвен serverless: Lambda с API Gateway, воркфлоу Step Functions, правила EventBridge, одно-табличный дизайн DynamoDB, оптимизация холодного старта и шаблоны SAM.

---

## Мониторинг

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
Полный стек обсервабильности: настройка Prometheus, правила записи, алерты на основе SLO, дашборды Grafana с переменными, маршрутизация Alertmanager и Thanos для долгосрочного хранения.

### [Стек логирования](skills/监控告警/logging-stack.md)
Централизованное логирование: деплой Grafana Loki, отправка логов Fluent Bit/Vector, запросы LogQL, политики ILM Elasticsearch, структурированное логирование и алерты на основе логов.

### [Реагирование на инциденты](skills/监控告警/incident-response.md)
Управление инцидентами: политики эскалации PagerDuty, шаблоны ранбуков, формат постмортема, определения серьёзности, шаблоны коммуникации и трекинг SLO-бюджета ошибок.

---

## Участие

Приветствуются вклады! Смотрите [CONTRIBUTING.md](CONTRIBUTING.md) для рекомендаций.

**Как внести вклад:**

1. Сделайте форк репозитория
2. Создайте ветку для фичи
3. Добавьте или улучшите файл навыка
4. Отправьте Pull Request

**Требования к навыку:**

- 150+ строк на файл
- Должны быть все 5 секций
- Продакшн-шаблоны кода

---

## Также посмотрите

Другие потрясающие коллекции, которые могут быть полезны:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — Самостоятельно размещённое ПО
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Ресурсы Kubernetes
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Экосистема Docker
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Модули и инструменты Terraform
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Экосистема Prometheus
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — Ресурсы AWS
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — Site Reliability Engineering
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Чарты и инструменты Helm
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — Инструменты CI/CD-пайплайнов
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Дашборды и плагины Grafana
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — Ресурсы управления инцидентами
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Фреймворки и инструменты serverless

---

## Лицензия

[Лицензия MIT](LICENSE) — Copyright (c) 2026 liangzhengtao

---

<p align="center">
  Сделано с заботой для DevOps-сообщества<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">Поставьте звезду этому репозиторию</a>, если он оказался полезен
</p>
