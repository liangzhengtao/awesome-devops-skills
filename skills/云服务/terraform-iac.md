# Infrastructure as Code with Terraform / Terraform 基础设施即代码

> Terraform modules, state management, workspaces, testing, and production patterns for managing cloud infrastructure at scale.

## When to Use / 何时使用

Use this skill when:

- Writing Terraform configurations for new infrastructure
- Designing reusable Terraform modules
- Setting up remote state with locking (S3 + DynamoDB, GCS, Terraform Cloud)
- Managing multiple environments with workspaces or directory layout
- Implementing Terraform in CI/CD pipelines
- Refactoring existing Terraform code to follow best practices
- Debugging Terraform state drift and plan failures
- Migrating resources between Terraform modules or states

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Terraform Project Structure                    │
│                                                                   │
│  infra/                                                          │
│  ├── modules/                # Reusable modules                  │
│  │   ├── networking/                                             │
│  │   │   ├── main.tf                                             │
│  │   │   ├── variables.tf                                        │
│  │   │   ├── outputs.tf                                          │
│  │   │   └── versions.tf                                         │
│  │   ├── compute/                                                │
│  │   └── database/                                               │
│  ├── environments/           # Environment-specific configs      │
│  │   ├── staging/                                                │
│  │   │   ├── main.tf        # Calls modules                     │
│  │   │   ├── terraform.tfvars                                    │
│  │   │   └── backend.tf     # Remote state config               │
│  │   └── production/                                             │
│  │       ├── main.tf                                             │
│  │       ├── terraform.tfvars                                    │
│  │       └── backend.tf                                          │
│  └── global/                 # Global resources                  │
│      ├── iam/                                                    │
│      ├── dns/                                                    │
│      └── monitoring/                                             │
│                                                                   │
│  State Backend: S3 + DynamoDB (locking)                         │
│  CI/CD: GitHub Actions / GitLab CI with terraform-plan/apply    │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Production Module Structure

```hcl
# modules/networking/main.tf
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.tags, {
    Name = "${var.name_prefix}-vpc"
  })
}

resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.this.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone = var.availability_zones[count.index]

  tags = merge(var.tags, {
    Name                              = "${var.name_prefix}-private-${var.availability_zones[count.index]}"
    "kubernetes.io/role/internal-elb" = "1"
  })
}

resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.this.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 100)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true

  tags = merge(var.tags, {
    Name                     = "${var.name_prefix}-public-${var.availability_zones[count.index]}"
    "kubernetes.io/role/elb" = "1"
  })
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
  tags   = merge(var.tags, { Name = "${var.name_prefix}-igw" })
}

resource "aws_nat_gateway" "this" {
  count         = var.single_nat_gateway ? 1 : length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  tags          = merge(var.tags, { Name = "${var.name_prefix}-nat-${count.index}" })
}
```

```hcl
# modules/networking/variables.tf
variable "name_prefix" {
  description = "Prefix for all resource names"
  type        = string
  validation {
    condition     = length(var.name_prefix) > 0 && length(var.name_prefix) <= 32
    error_message = "name_prefix must be 1-32 characters."
  }
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "Must be a valid CIDR block."
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  validation {
    condition     = length(var.availability_zones) >= 2
    error_message = "At least 2 AZs required for HA."
  }
}

variable "single_nat_gateway" {
  description = "Use a single NAT gateway (cost saving for non-prod)"
  type        = bool
  default     = false
}

variable "tags" {
  description = "Common tags for all resources"
  type        = map(string)
  default     = {}
}
```

```hcl
# modules/networking/outputs.tf
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.this.id
}

output "private_subnet_ids" {
  description = "List of private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "nat_gateway_ips" {
  description = "NAT gateway public IPs"
  value       = aws_eip.nat[*].public_ip
}
```

```hcl
# modules/networking/versions.tf
terraform {
  required_version = ">= 1.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

### 2. Remote State Backend with Locking

```hcl
# environments/production/backend.tf
terraform {
  backend "s3" {
    bucket         = "my-company-terraform-state"
    key            = "production/networking/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-state-lock"
    kms_key_id     = "alias/terraform-state"
  }
}
```

```bash
# One-time setup: create state backend
aws s3api create-bucket \
  --bucket my-company-terraform-state \
  --region us-east-1

aws s3api put-bucket-versioning \
  --bucket my-company-terraform-state \
  --versioning-configuration Status=Enabled

aws s3api put-bucket-encryption \
  --bucket my-company-terraform-state \
  --server-side-encryption-configuration '{
    "Rules": [{"ApplyServerSideEncryptionByDefault": {"SSEAlgorithm": "aws:kms"}}]
  }'

aws dynamodb create-table \
  --table-name terraform-state-lock \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### 3. Environment Composition

```hcl
# environments/production/main.tf
module "networking" {
  source = "../../modules/networking"

  name_prefix        = "prod"
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
  single_nat_gateway = false

  tags = local.common_tags
}

module "database" {
  source = "../../modules/database"

  name_prefix        = "prod"
  engine             = "aurora-postgresql"
  engine_version     = "15.4"
  instance_class     = "db.serverless"
  instances          = 2
  vpc_id             = module.networking.vpc_id
  subnet_ids         = module.networking.private_subnet_ids
  backup_retention   = 30
  deletion_protection = true

  tags = local.common_tags
}

module "compute" {
  source = "../../modules/compute"

  name_prefix     = "prod"
  vpc_id          = module.networking.vpc_id
  subnet_ids      = module.networking.private_subnet_ids
  cluster_size    = 3
  instance_type   = "t3.medium"

  database_endpoint = module.database.endpoint
  database_secret   = module.database.secret_arn

  tags = local.common_tags
}

locals {
  common_tags = {
    Environment = "production"
    Project     = "my-app"
    ManagedBy   = "terraform"
    Team        = "platform"
  }
}
```

### 4. CI/CD Pipeline for Terraform

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
    paths: ["infra/**"]
  push:
    branches: [main]
    paths: ["infra/**"]

permissions:
  contents: read
  pull-requests: write
  id-token: write

env:
  TF_VERSION: "1.7.5"
  AWS_REGION: "us-east-1"

jobs:
  plan:
    name: Plan
    runs-on: ubuntu-latest
    strategy:
      matrix:
        env: [staging, production]
    defaults:
      run:
        working-directory: infra/environments/${{ matrix.env }}
    steps:
      - uses: actions/checkout@v4

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}

      - run: terraform init -backend=true
      - run: terraform validate
      - id: plan
        run: terraform plan -no-color -out=plan.tfplan 2>&1 | tee plan.txt

      - name: Comment PR with plan
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const plan = fs.readFileSync('${{ matrix.env }}/plan.txt', 'utf8');
            const body = `### Terraform Plan — ${{ matrix.env }}
            \`\`\`hcl\n${plan.slice(0, 60000)}\n\`\`\``;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body
            });

  apply:
    name: Apply
    needs: plan
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}
      - uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      - working-directory: infra/environments/production
        run: |
          terraform init
          terraform apply -auto-approve
```

### 5. Testing with Terratest

```go
// test/vpc_test.go
package test

import (
	"testing"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestVpcModule(t *testing.T) {
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		TerraformDir: "../modules/networking",
		Vars: map[string]interface{}{
			"name_prefix":        "test",
			"vpc_cidr":           "10.99.0.0/16",
			"availability_zones": []string{"us-east-1a", "us-east-1b"},
			"single_nat_gateway": true,
			"tags": map[string]string{
				"Environment": "test",
			},
		},
	})
	defer terraform.Destroy(t, terraformOptions)

	terraform.InitAndApply(t, terraformOptions)

	vpcId := terraform.Output(t, terraformOptions, "vpc_id")
	assert.NotEmpty(t, vpcId)

	privateSubnets := terraform.OutputList(t, terraformOptions, "private_subnet_ids")
	assert.Equal(t, 2, len(privateSubnets))
}
```

## Best Practices / 最佳实践

1. **Use remote state with locking** — S3 + DynamoDB for AWS, GCS for GCP. Never use local state for shared infrastructure.
2. **Separate state files per environment** — `production/networking/terraform.tfstate` vs `staging/networking/terraform.tfstate`. Blast radius isolation.
3. **Use `terraform plan` in CI** — always plan before apply. Post the plan output as a PR comment for review.
4. **Pin provider versions** — `required_providers` with `~>` constraint prevents surprise breaking changes.
5. **Use modules for DRY** — extract reusable components (VPC, ECS, RDS) into modules. Version them.
6. **Set `prevent_destroy` on critical resources** — databases, S3 buckets, DNS zones. Prevents accidental deletion.
7. **Use `moved` blocks for refactoring** — rename resources without destroying and recreating them.
8. **Import existing resources** — `terraform import` existing infrastructure into state before managing it.
9. **Use `terraform fmt` and `terraform validate`** — enforce in CI. Consistent formatting prevents noisy diffs.
10. **Tag everything** — consistent tags enable cost allocation, access control, and automated cleanup.

## Pitfalls / 常见陷阱

1. **State file conflicts** — without DynamoDB locking, concurrent `terraform apply` corrupts state. Always enable locking.
2. **Sensitive data in state** — Terraform state contains all resource attributes, including passwords. Encrypt at rest and restrict access.
3. **`terraform apply` without plan** — always run plan first. Auto-approve in CI should only apply a previously-reviewed plan.
4. **Module version pinning** — without version constraints, a module update can break all environments. Use semantic versioning.
5. **Circular dependencies** — module A depends on module B output which depends on module A. Restructure with data sources.
6. **Destroying the state backend** — if you `terraform destroy` the S3 bucket holding state, all managed resources become unmanageable.
7. **Large state files** — state files over 100MB slow plan/apply. Split into separate state files by layer (network, compute, data).
8. **Provider aliasing mistakes** — multi-region deployments need aliased providers. Forgetting the alias applies changes to the wrong region.
9. **Data source caching** — `data.aws_ami` caches in state. Add lifecycle or use `terraform refresh` to get latest AMI.
10. **Terraform Cloud costs** — Terraform Cloud charges per resource under management. Self-hosted backends (S3) are cheaper at scale.

---

## 中文版本

### 使用场景

- 为新基础设施编写 Terraform 配置
- 设计可复用的 Terraform 模块
- 设置带锁的远程状态存储（S3 + DynamoDB）
- 使用 workspace 或目录布局管理多环境
- 在 CI/CD 流水线中集成 Terraform
- 调试状态漂移和 plan 失败

### 核心步骤

1. **模块化设计** — 将 VPC、ECS、RDS 等抽取为独立模块，每个模块包含 main.tf、variables.tf、outputs.tf、versions.tf
2. **远程状态 + 锁** — 使用 S3 + DynamoDB 实现状态远程存储和并发锁，启用 KMS 加密
3. **环境组合** — 每个环境（staging/production）调用相同模块但传入不同参数，状态文件按环境隔离
4. **CI/CD 集成** — PR 时自动 `terraform plan` 并将结果评论到 PR，main 分支合并后自动 `terraform apply`
5. **测试** — 使用 Terratest 编写 Go 测试验证模块输出

### 模板说明

- 网络模块 — VPC + 公私子网 + NAT Gateway + Internet Gateway 的完整模块
- 远程状态 — S3 + DynamoDB 后端配置和一次性初始化脚本
- 环境组合 — production 环境如何组合 networking、database、compute 模块
- CI 流水线 — GitHub Actions 中的 plan/apply 流水线，支持 PR 评论

### 常见陷阱

1. **状态文件冲突** — 没有 DynamoDB 锁时并发 `terraform apply` 会损坏状态
2. **状态中的敏感数据** — Terraform 状态包含所有资源属性（包括密码），需加密存储并限制访问
3. **不先 plan 就 apply** — CI 中自动 apply 应仅针对已审查的 plan
4. **销毁状态后端** — `terraform destroy` 持有状态的 S3 bucket 会导致所有托管资源不可管理
5. **Provider 别名遗漏** — 多区域部署需要 aliased provider，遗漏别名会将变更应用到错误区域
