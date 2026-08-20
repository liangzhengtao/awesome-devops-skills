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

> **소방 활동을 멈추세요. 인프라를 자동화하는 12가지 AI 스킬.**

DevOps 및 인프라 자동화를 위한 프로덕션 AI 스킬 모음. 각 스킬 파일은 AI 에이전트가 로드하여 프로덕션 시스템을 구축, 배포, 모니터링, 운영할 수 있도록 돕는 독립형 지식 모듈입니다.

---

## 목차

- [스킬 개요](#skills-overview)
- [빠른 시작](#quick-start)
- [CI/CD](#cicd)
- [컨테이너화](#containerization)
- [클라우드](#cloud)
- [모니터링](#monitoring)
- [기여](#contributing)
- [관련 프로젝트](#see-also)

---

## 스킬 개요

| # | 카테고리 | 스킬 | 설명 |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md) | 매트릭스 빌드, OIDC, 재사용 가능한 워크플로우, 캐싱 |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | 러너, DAG 파이프라인, 환경, 보안 스캐닝 |
| 3 | CI/CD | [배포 전략](skills/CI-CD/deployment-strategies.md) | 블루-그린, 카나리아, 롤링, Argo Rollouts |
| 4 | 컨테이너화 | [Docker 프로덕션](skills/容器化/docker-production.md) | 멀티스테이지 빌드, 보안 강화, 이미지 최적화 |
| 5 | 컨테이너화 | [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md) | 배포, RBAC, HPA, 네트워킹, 트러블슈팅 |
| 6 | 컨테이너화 | [Helm Charts](skills/容器化/helm-charts.md) | 차트 개발, 템플릿, 테스팅, 저장소 |
| 7 | 클라우드 | [AWS Essentials](skills/云服务/aws-essentials.md) | IAM, VPC, ECS, RDS, S3, 비용 최적화 |
| 8 | 클라우드 | [Terraform IaC](skills/云服务/terraform-iac.md) | 모듈, 상태 관리, 워크스페이스, CI/CD 통합 |
| 9 | 클라우드 | [서버리스 패턴](skills/云服务/serverless-patterns.md) | Lambda, API Gateway, Step Functions, EventBridge |
| 10 | 모니터링 | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | 익스포터, PromQL, 대시보드, SLO 기반 알림 |
| 11 | 모니터링 | [로깅 스택](skills/监控告警/logging-stack.md) | ELK, Loki, Fluentd, Vector, LogQL, 보존 |
| 12 | 모니터링 | [장애 대응](skills/监控告警/incident-response.md) | 온콜, 런북, 사후 분석, 에스컬레이션 |

---

## 빠른 시작

### AI 에이전트 사용자를 위한 가이드

1. 위 표에서 작업에 맞는 **스킬을 선택하세요**
2. **스킬 파일을** AI 에이전트의 컨텍스트에 **로드하세요**
3. **템플릿을 적용하세요** — 각 스킬에는 프로덕션 코드가 포함되어 있습니다

### 개발자를 위한 가이드

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### 스킬 파일 구조

각 스킬은 일관된 형식을 따릅니다:

```
┌─────────────────────────────────┐
│  사용 시점                       │  ← 명확한 트리거 조건
│  아키텍처                        │  ← ASCII 다이어그램
│  코드 템플릿                     │  ← YAML/HCL/Dockerfile
│  검증된 패턴                     │  ← 실전 추천사항
│  주의사항                        │  ← 피해야 할 실수
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions Advanced](skills/CI-CD/github-actions-advanced.md)
GitHub Actions 고급 패턴: 매트릭스 빌드, OIDC 인증, 재사용 가능한 워크플로우, 캐싱 전략, 셀프호스티드 러너, 시맨틱 릴리스 자동화.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
프로덕션 GitLab 파이프라인: DAG 최적화, 자동 스케일링 러너, 리뷰 앱, 보안 스캐닝 템플릿, 다중 환경 배포.

### [배포 전략](skills/CI-CD/deployment-strategies.md)
무중단 배포 패턴: Argo Rollouts를 활용한 블루-그린, 트래픽 분할 및 자동 분석을 활용한 카나리아, 적절한 헬스 체크가 포함된 롤링 업데이트.

---

## 컨테이너화

### [Docker 프로덕션](skills/容器化/docker-production.md)
프로덕션 Docker 패턴: 멀티스테이지 빌드 (45MB 이미지), 비루트 컨테이너, Trivy를 활용한 보안 스캐닝, 멀티 아키텍처 빌드, 프로덕션용 Docker Compose.

### [Kubernetes Essentials](skills/容器化/kubernetes-essentials.md)
K8s 프로덕션 패턴: 토포로지 스프레드가 포함된 배포, 최소 권한 RBAC, 커스텀 메트릭이 포함된 HPA, NetworkPolicies, 완전한 트러블슈팅 명령어.

### [Helm Charts](skills/容器化/helm-charts.md)
Helm 검증된 패턴: 모듈형 차트 구조, 템플릿 헬퍼, 다중 환경 값, `--atomic` 배포, 차트 테스팅, OCI 레지스트리 배포.

---

## 클라우드

### [AWS Essentials](skills/云服务/aws-essentials.md)
AWS 프로덕션 아키텍처: 다중 AZ가 포함된 VPC, OIDC를 활용한 최소 권한 IAM, ECS Fargate 서비스, Aurora Serverless, S3 보안, 예산을 활용한 비용 최적화.

### [Terraform IaC](skills/云服务/terraform-iac.md)
대규모 Terraform: 재사용 가능한 모듈, 잠금이 포함된 원격 상태, 환경 분리, plan/apply를 활용한 CI/CD, Terratest, 상태 마이그레이션 전략.

### [서버리스 패턴](skills/云服务/serverless-patterns.md)
이벤트 기반 서버리스: API Gateway가 포함된 Lambda, Step Functions 워크플로우, EventBridge 규칙, DynamoDB 단일 테이블 설계, 콜드 스타트 최적화, SAM 템플릿.

---

## 모니터링

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
완전한 옵저버빌리티 스택: Prometheus 설정, 레코딩 규칙, SLO 기반 알림, 변수가 포함된 Grafana 대시보드, Alertmanager 라우팅, 장기 저장을 위한 Thanos.

### [로깅 스택](skills/监控告警/logging-stack.md)
集中式 로깅: Grafana Loki 배포, Fluent Bit/Vector 로그 전송, LogQL 쿼리, Elasticsearch ILM 정책, 구조화된 로깅, 로그 기반 알림.

### [장애 대응](skills/监控告警/incident-response.md)
장애 관리: PagerDuty 에스컬레이션 정책, 런북 템플릿, 사후 분석 형식, 심각도 정의, 커뮤니케이션 템플릿, SLO 에러 예산 추적.

---

## 기여

기여를 환영합니다! [CONTRIBUTING.md](CONTRIBUTING.md)에서 가이드라인을 확인하세요.

**기여 방법:**

1. 저장소를 포크하세요
2. 기능 브랜치를 만드세요
3. 스킬 파일을 추가하거나 개선하세요
4. Pull Request를 제출하세요

**스킬 요구사항:**

- 파일당 150줄 이상
- 모든 5개 섹션 포함 필수
- 프로덕션 코드 템플릿

---

## 관련 프로젝트

유용할 수 있는 다른 멋진 컬렉션:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — 셀프호스티드 소프트웨어
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetes 리소스
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Docker 생태계
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraform 모듈 및 도구
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheus 생태계
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWS 리소스
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — 사이트 신뢰성 엔지니어링
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm 차트 및 도구
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CD 파이프라인 도구
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafana 대시보드 및 플러그인
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — 장애 관리 리소스
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — 서버리스 프레임워크 및 도구

---

## 라이선스

[MIT 라이선스](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  DevOps 커뮤니티를 위해 정성껏 만들었습니다<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">이 저장소에 스타를 주세요</a> 도움이 되셨다면
</p>
