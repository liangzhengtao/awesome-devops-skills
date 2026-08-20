[English](README.md) | [中文](README.zh.md) | [日本語](README.ja.md) | [Français](README.fr.md) | [Español](README.es.md) | [العربية](README.ar.md) | [한국어](README.ko.md) | [Português](README.pt.md) | [Русский](README.ru.md) | [Deutsch](README.de.md)

<div dir="rtl" align="center">

<img src=".banner.svg" width="100%" alt="banner">

</div>


# مهارات DevOps الرائعة

[![CI](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml/badge.svg)](https://github.com/liangzhengtao/awesome-devops-skills/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Awesome](https://awesome.re/badge.svg)](https://awesome.re)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

> **توقف عن إطفاء الحرائق. 12 مهارة AI لأتمتة بنيتك التحتية.**

مجموعة منتقاة من مهارات AI الإنتاجية لDevOps وأتمتة البنية التحتية. كل ملف مهارة هو وحدة معرفية مستقلة يمكن لوكيل AI تحميلها لمساعدتك في بناء ونشر ومراقبة وتشغيل الأنظمة الإنتاجية.

---

## جدول المحتويات

- [نظرة عامة على المهارات](#skills-overview)
- [البدء السريع](#quick-start)
- [CI/CD](#cicd)
- [الحاويات](#containerization)
- [السحابة](#cloud)
- [المراقبة](#monitoring)
- [المساهمة](#contributing)
- [انظر أيضًا](#see-also)

---

## نظرة عامة على المهارات

| # | الفئة | المهارة | الوصف |
|---|----------|-------|-------------|
| 1 | CI/CD | [GitHub Actions المتقدم](skills/CI-CD/github-actions-advanced.md) | بناء المصفوفة، OIDC، سير العمل القابل لإعادة الاستخدام، التخزين المؤقت |
| 2 | CI/CD | [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md) | العوامل، خطوط أنابيب DAG، البيئات، فحص الأمان |
| 3 | CI/CD | [استراتيجيات النشر](skills/CI-CD/deployment-strategies.md) | أزرق-أخضر، التدريج، التحديث المتداول، Argo Rollouts |
| 4 | الحاويات | [Docker للإنتاج](skills/容器化/docker-production.md) | بناء متعدد المراحل، تقوية الأمان، تحسين الصور |
| 5 | الحاويات | [أساسيات Kubernetes](skills/容器化/kubernetes-essentials.md) | النشر، RBAC، HPA، الشبكات، استكشاف الأخطاء |
| 6 | الحاويات | [Helm Charts](skills/容器化/helm-charts.md) | تطوير الرسم البياني، القوالب، الاختبار، المستودعات |
| 7 | السحابة | [أساسيات AWS](skills/云服务/aws-essentials.md) | IAM، VPC، ECS، RDS، S3، تحسين التكلفة |
| 8 | السحابة | [Terraform IaC](skills/云服务/terraform-iac.md) | الوحدات، إدارة الحالة، مساحات العمل، تكامل CI/CD |
| 9 | السحابة | [أنماط بدون خادم](skills/云服务/serverless-patterns.md) | Lambda، API Gateway، Step Functions، EventBridge |
| 10 | المراقبة | [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md) | المُصدّرات، PromQL، لوحات التحكم، تنبيهات SLO |
| 11 | المراقبة | [مجموعة التسجيل](skills/监控告警/logging-stack.md) | ELK، Loki، Fluentd، Vector، LogQL، الاحتفاظ |
| 12 | المراقبة | [الاستجابة للحوادث](skills/监控告警/incident-response.md) | المناوبة، كتيبات التشغيل، ما بعد الحوادث، التصعيد |

---

## البدء السريع

### لمستخدمي وكلاء AI

1. **اختر مهارة** من الجدول أعلاه بناءً على مهمتك
2. **حمّل ملف المهارة** في سياق وكيل AI الخاص بك
3. **طبّق القوالب** — كل مهارة تحتوي على كود إنتاجي

### للمطورين

```bash
# Clone the repository
git clone https://github.com/liangzhengtao/awesome-devops-skills.git
cd awesome-devops-skills

# Browse skills
ls skills/*/

# Use in your project
cp skills/CI-CD/github-actions-advanced.md .github/workflows/
```

### هيكل ملف المهارة

كل مهارة تتبع تنسيقًا متسقًا:

```
┌─────────────────────────────────┐
│  متى تُستخدم                    │  ← شروط تشغيل واضحة
│  الهندسة المعمارية               │  ← مخططات ASCII
│  قوالب الكود                     │  ← YAML/HCL/Dockerfile
│  الأنماط المُجربة                │  ← توصيات مُجربة
│  الأخطاء الشائعة                 │  ← أخطاء يجب تجنبها
└─────────────────────────────────┘
```

---

## CI/CD

### [GitHub Actions المتقدم](skills/CI-CD/github-actions-advanced.md)
أنماط متقدمة لـ GitHub Actions: بناء المصفوفة، مصادقة OIDC، سير العمل القابل لإعادة الاستخدام، استراتيجيات التخزين المؤقت، العوامل المستضافة ذاتيًا، وأتمتة النشر الدلالي.

### [GitLab CI/CD](skills/CI-CD/gitlab-cicd.md)
خطوط أنابيب GitLab إنتاجية: تحسين DAG، عوامل تحجيم تلقائي، تطبيقات المراجعة، قوالب فحص الأمان، ونشر متعدد البيئات.

### [استراتيجيات النشر](skills/CI-CD/deployment-strategies.md)
أنماط نشر بدون توقف: أزرق-أخضر مع Argo Rollouts، التدريج مع تقسيم حركة المرور والتحليل التلقائي، التحديثات المتداول مع فحوصات الحالة الصحية المناسبة.

---

## الحاويات

### [Docker للإنتاج](skills/容器化/docker-production.md)
أنماط Docker إنتاجية: بناء متعدد المراحل (صور 45MB)، حاويات غير جذرية، فحص الأمان مع Trivy، بناء متعدد المعماريات، وDocker Compose للإنتاج.

### [أساسيات Kubernetes](skills/容器化/kubernetes-essentials.md)
أنماط K8s إنتاجية: النشر مع توزيع الطبوغرافيا، RBAC بأقل امتياز، HPA مع مقاييس مخصصة، NetworkPolicies، وأوامر استكشاف الأخطاء الكاملة.

### [Helm Charts](skills/容器化/helm-charts.md)
أنماط Helm مُجربة: هيكل رسم بياني modular، مساعدات القوالب، قيم متعددة البيئات، نشر `--atomic`، اختبار الرسم البياني، ونشر سجل OCI.

---

## السحابة

### [أساسيات AWS](skills/云服务/aws-essentials.md)
بنية AWS إنتاجية: VPC مع متعددة AZ، IAM بأقل امتياز مع OIDC، خدمات ECS Fargate، Aurora Serverless، أمان S3، وتحسين التكلفة مع الميزانيات.

### [Terraform IaC](skills/云服务/terraform-iac.md)
Terraform على نطاق واسع: وحدات قابلة لإعادة الاستخدام، حالة بعيدة مع قفل، فصل البيئات، CI/CD مع plan/apply، Terratest، واستراتيجيات ترحيل الحالة.

### [أنماط بدون خادم](skills/云服务/serverless-patterns.md)
بدون خادم مُدار بالأحداث: Lambda مع API Gateway، سير عمل Step Functions، قواعد EventBridge، تصميم جدول DynamoDB المفرد، تحسين البارد، وقوالب SAM.

---

## المراقبة

### [Prometheus + Grafana](skills/监控告警/prometheus-grafana.md)
مجموعة رصد كاملة: تكوين Prometheus، قواعد التسجيل، تنبيهات قائمة على SLO، لوحات Grafana مع المتغيرات، توجيه Alertmanager، وThanos للتخزين طويل المدى.

### [مجموعة التسجيل](skills/监控告警/logging-stack.md)
تسجيل مركزي: نشر Grafana Loki، شحن السجلات Fluent Bit/Vector، استعلامات LogQL، سياسات ILM لـ Elasticsearch، التسجيل الهيكلي، وتنبيهات مبنية على السجلات.

### [الاستجابة للحوادث](skills/监控告警/incident-response.md)
إدارة الحوادث: سياسات تصعيد PagerDuty، قوالب كتيبات التشغيل، تنسيق ما بعد الحوادث، تعريفات الخطورة، قوالب الاتصال، وتتبع ميزانية الخطأ SLO.

---

## المساهمة

نرحب بالمسايرات! راجع [CONTRIBUTING.md](CONTRIBUTING.md) للإرشادات.

**كيف تساهم:**

1. قم بعمل Fork للمستودع
2. أنشئ فرع ميزة
3. أضف أو حسّن ملف مهارة
4. قدّم طلب سحب

**متطلبات المهارة:**

- 150+ سطرًا لكل ملف
- يجب أن تتضمن جميع الأقسام الخمسة
- قوالب كود إنتاجية

---

## انظر أيضًا

مجموعات رائعة أخرى قد تجدها مفيدة:

- **[awesome-selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)** — برمجيات مستضافة ذاتيًا
- **[awesome-kubernetes](https://github.com/ramitsurana/awesome-kubernetes)** — موارد Kubernetes
- **[awesome-docker](https://github.com/veggiemonk/awesome-docker)** — نظام Docker البيئي
- **[awesome-terraform](https://github.com/shuaibiyy/awesome-terraform)** — وحدات وأدوات Terraform
- **[awesome-prometheus](https://github.com/roaldnefs/awesome-prometheus)** — نظام Prometheus البيئي
- **[awesome-aws](https://github.com/donnemartin/awesome-aws)** — موارد AWS
- **[awesome-sre](https://github.com/dastergon/awesome-sre)** — هندسة موثوقية الموقع
- **[awesome-helm](https://github.com/cdwv/awesome-helm)** — رسوم Helm البيئية
- **[awesome-pipeline](https://github.com/pditommaso/awesome-pipeline)** — أدوات خطوط أنابيب CI/CD
- **[awesome-grafana](https://github.com/iamchucky/awesome-grafana)** — لوحات وإضافات Grafana
- **[awesome-incidents](https://github.com/jonah-jones/awesome-incidents)** — موارد إدارة الحوادث
- **[awesome-serverless](https://github.com/anaibol/awesome-serverless)** — أطر وأدوات بدون خادم

---

## الترخيص

[ترخيص MIT](LICENSE) - حقوق النشر (c) 2026 liangzhengtao

---

<p dir="rtl" align="center">
  صُنع بعناية لمجتمع DevOps<br>
  <a href="https://github.com/liangzhengtao/awesome-devops-skills">أعطِ نجمة لهذا المستودع</a> إذا وجدته مفيدًا
</p>
