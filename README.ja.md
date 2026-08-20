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

> **消防活動をやめよう。インフラを自動化する12のAIスキル。**

DevOpsとインフラ自動化のためのプロダクションAIスキルコレクション。各スキルファイルは独立したナレッジモジュールで、AIエージェントが読み込んで、本番環境の構築、デプロイ、監視、運用を支援します。

---

## 目次

- [スキル一覧](#スキル一覧)
- [クイックスタート](#クイックスタート)
- [CI/CD](#cicd)
- [コンテナ化](#コンテナ化)
- [クラウド](#クラウド)
- [モニタリング](#モニタリング)
- [コントリビュート](#コントリビュート)
- [関連項目](#関連項目)

---

## スキル一覧

| # | カテゴリ | スキル | 説明 |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions アドバンスド](skills/CI-CD/github-actions-advanced.md) | マトリックスビルド、OIDC、再利用可能ワークフロー、キャッシュ |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | ランナー、DAGパイプライン、環境管理、セキュリティスキャン |
| 3 | CI/CD | [デプロイ戦略](skills/CI-CD/deployment-strategies.md) | ブルー/グリーン、カナリー、ローリング、Argo Rollouts |
| 4 | コンテナ化 | [Docker本番運用](skills/容器化/docker-production.md) | マルチステージビルド、セキュリティ強化、イメージ最適化 |
| 5 | コンテナ化 | [Kubernetes要点](skills/容器化/kubernetes-essentials.md) | Deployment、RBAC、HPA、ネットワーキング、トラブルシューティング |
| 6 | コンテナ化 | [Helm Charts](skills/容器化/helm-charts.md) | Chart開発、テンプレート、テスト、リポジトリ管理 |
| 7 | クラウド | [AWS要点](skills/云服务/aws-essentials.md) | IAM、VPC、ECS、RDS、S3、コスト最適化 |
| 8 | クラウド | [Terraform IaC](skills/云服务/terraform-iac.md) | モジュール、ステート管理、ワークスペース、CI/CD統合 |
| 9 | クラウド | [Serverlessパターン](skills/云服务/serverless-patterns.md) | Lambda、API Gateway、Step Functions、EventBridge |
| 10 | モニタリング | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | Exporter、PromQL、ダッシュボード、SLOベースアラート |
| 11 | モニタリング | [ログ基盤](skills/监控告警/logging-stack.md) | ELK、Loki、Fluentd、Vector、LogQL、リテンション |
| 12 | モニタリング | [インシデント対応](skills/监控告警/incident-response.md) | オンコール、Runbook、事後レビュー、エスカレーション |

---

## クイックスタート

### AIエージェントユーザー向け

1. 上記のテーブルからタスクに合った**スキルを選択**
2. スキルファイルをAIエージェントのコンテキストに**読み込み**
3. **テンプレートを適用** — 各スキルにプロダクションコードが含まれています

### 開発者向け

```bash
# リポジトリをクローン
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# スキルを閲覧
ls skills/*/

# プロジェクトで使用
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### スキルファイル構造

各スキルは統一フォーマットに従います：

```
┌─────────────────────────────────┐
│  使用タイミング                  │  ← 明確なトリガー条件
│  アーキテクチャ                  │  ← ASCII 図
│  コードテンプレート              │  ← YAML/HCL/Dockerfile
│  ベストプラクティス              │  ← 実績のある推奨事項
│  落とし穴                        │  ← 避けるべきよくあるミス
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions アドバンスド](skills/CI-CD/github-actions-advanced.md)
GitHub Actionsの高度なパターン：マトリックスビルド、OIDC認証、再利用可能ワークフロー、キャッシュ戦略、セルフホストランナー、セマンティックリリース自動化。

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
本番向けGitLabパイプライン：DAG最適化、自動スケーリングランナー、Review Apps、セキュリティスキャンテンプレート、マルチ環境デプロイ。

### [デプロイ戦略](skills/CI-CD/deployment-strategies.md)
ゼロダウンタイムデプロイパターン：Argo Rolloutsによるブルー/グリーン、トラフィック分割と自動分析によるカナリーリリース、ヘルスチェック付きローリングアップデート。

---

## コンテナ化

### [Docker本番運用](skills/容器化/docker-production.md)
本番向けDockerパターン：マルチステージビルド（45MBイメージ）、非rootコンテナ、Trivyセキュリティスキャン、マルチアーキテクチャビルド、プロダクション向けDocker Compose。

### [Kubernetes要点](skills/容器化/kubernetes-essentials.md)
K8s本番パターン：トポロジ分散デプロイ、RBAC最小権限、カスタムメトリクスHPA、NetworkPolicy、包括的なトラブルシューティングコマンド。

### [Helm Charts](skills/容器化/helm-charts.md)
Helmベストプラクティス：モジュラーChart構造、テンプレートヘルパー、マルチ環境values、`--atomic`デプロイ、Chartテスト、OCIレジストリ公開。

---

## クラウド

### [AWS要点](skills/云服务/aws-essentials.md)
AWS本番アーキテクチャ：マルチAZ VPC、OIDC IAM最小権限、ECS Fargateサービス、Aurora Serverless、S3セキュリティ、予算ベースコスト最適化。

### [Terraform IaC](skills/云服务/terraform-iac.md)
大規模Terraform：再利用可能モジュール、ロック付きリモートステート、環境分離、CI/CD plan/apply、Terratest、ステート移行戦略。

### [Serverlessパターン](skills/云服务/serverless-patterns.md)
イベント駆動Serverless：Lambda + API Gateway、Step Functionsワークフロー、EventBridgeルール、DynamoDB単一テーブル設計、コールドスタート最適化、SAMテンプレート。

---

## モニタリング

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
完全なオブザーバビリティスタック：Prometheus設定、Recording Rules、SLOベースアラート、変数付きGrafanaダッシュボード、Alertmanagerルーティング、Thanos長期保存。

### [ログ基盤](skills/监控告警/logging-stack.md)
集中ログ管理：Grafana Lokiデプロイ、Fluent Bit/Vectorログ転送、LogQLクエリ、Elasticsearch ILMポリシー、構造化ログ、ログベースアラート。

### [インシデント対応](skills/监控告警/incident-response.md)
インシデント管理：PagerDutyエスカレーションポリシー、Runbookテンプレート、事後レビュー形式、深刻度定義、コミュニケーションテンプレート、SLOエラーバジェット追跡。

---

## コントリビュート

コントリビューションを歓迎します！ガイドラインは [CONTRIBUTING.md](CONTRIBUTING.md) をご覧ください。

**コントリビュートの方法：**

1. リポジトリをFork
2. 機能ブランチを作成
3. スキルファイルを追加または改善
4. Pull Requestを送信

**スキルの要件：**
- ファイルあたり150行以上
- 5つのセクションすべてを含むこと
- プロダクションコードテンプレート

---

## 関連項目

参考になるその他のawesomeコレクション：

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — セルフホストソフトウェア
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — Kubernetesリソース
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — Dockerエコシステム
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — Terraformモジュールとツール
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — Prometheusエコシステム
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — AWSリソース
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — サイト信頼性エンジニアリング
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — Helm Chartsとツール
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — CI/CDパイプラインツール
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — Grafanaダッシュボードとプラグイン
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — インシデント管理リソース
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — Serverlessフレームワークとツール

---

## ライセンス

[MIT License](LICENSE) - Copyright (c) 2026 liangzhengtao

---

<p align="center">
  DevOpsコミュニティのために心を込めて作りました<br>
  有用であれば<a href="https://github.com/liangzhengtao/awesome-devops-skills">Starを付けてください</a>
</p>
