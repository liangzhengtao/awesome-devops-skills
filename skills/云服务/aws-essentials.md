# AWS Essentials / AWS 核心服务

> Core AWS services for production workloads: IAM, VPC, EC2, ECS, S3, RDS, CloudFront, and cost optimization patterns.

## When to Use / 何时使用

Use this skill when:

- Designing AWS infrastructure for new applications
- Configuring IAM policies and roles with least privilege
- Setting up VPC networking with public/private subnets
- Deploying applications on ECS, EKS, or EC2
- Managing S3 buckets with proper security and lifecycle
- Provisioning RDS databases with high availability
- Optimizing AWS costs and reserved capacity
- Implementing multi-region architectures

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                       AWS Production Architecture                 │
│                                                                   │
│  CloudFront (CDN) ──► ALB ──► ECS Fargate Cluster               │
│       │                          │                                │
│       │                    ┌─────┴─────┐                          │
│       │                    │  Service   │                          │
│  S3 (static)              │  (tasks)   │                          │
│                            └─────┬─────┘                          │
│                                  │                                │
│  VPC (10.0.0.0/16)              │                                │
│  ├── Public Subnet (10.0.1.0/24)│  NAT Gateway                   │
│  │   └── ALB                    │                                │
│  ├── Private Subnet (10.0.2.0/24)                                │
│  │   └── ECS Tasks ────────────┤                                │
│  └── Isolated Subnet (10.0.3.0/24)                               │
│      └── RDS Multi-AZ          │                                │
│      └── ElastiCache            │                                │
│                                                                   │
│  Security: IAM Roles, Security Groups, NACLs, WAF               │
│  Monitoring: CloudWatch, X-Ray, CloudTrail                       │
│  Secrets: Secrets Manager, Parameter Store                       │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. VPC with Multi-AZ Subnets

```hcl
# vpc.tf
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "production-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  intra_subnets   = ["10.0.201.0/24", "10.0.202.0/24", "10.0.203.0/24"]

  enable_nat_gateway     = true
  single_nat_gateway     = false  # HA: one NAT GW per AZ
  one_nat_gateway_per_az = true
  enable_dns_hostnames   = true
  enable_dns_support     = true

  # Flow logs for security auditing
  enable_flow_log                      = true
  create_flow_log_cloudwatch_log_group = true
  create_flow_log_iam_role             = true
  flow_log_max_aggregation_interval    = 60

  tags = {
    Environment = "production"
    Terraform   = "true"
  }
}
```

### 2. IAM Policies (Least Privilege)

```hcl
# iam.tf
# Application task role — only what the app needs
resource "aws_iam_role" "app_task" {
  name = "my-app-task-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ecs-tasks.amazonaws.com"
      }
      Condition = {
        StringEquals = {
          "aws:SourceAccount" = data.aws_caller_identity.current.account_id
        }
      }
    }]
  })
}

# S3 access — scoped to specific bucket and prefix
resource "aws_iam_role_policy" "app_s3" {
  name = "s3-access"
  role = aws_iam_role.app_task.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "s3:GetObject",
          "s3:PutObject",
          "s3:DeleteObject"
        ]
        Resource = "${aws_s3_bucket.app_data.arn}/*"
      },
      {
        Effect   = "Allow"
        Action   = "s3:ListBucket"
        Resource = aws_s3_bucket.app_data.arn
        Condition = {
          StringLike = {
            "s3:prefix" = ["uploads/*", "exports/*"]
          }
        }
      }
    ]
  })
}

# Secrets Manager — scoped to specific secrets
resource "aws_iam_role_policy" "app_secrets" {
  name = "secrets-access"
  role = aws_iam_role.app_task.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = ["secretsmanager:GetSecretValue"]
      Resource = [
        aws_secretsmanager_secret.db_password.arn,
        aws_secretsmanager_secret.api_key.arn
      ]
    }]
  })
}

# OIDC provider for GitHub Actions
resource "aws_iam_openid_connect_provider" "github" {
  url             = "https://token.actions.githubusercontent.com"
  client_id_list  = ["sts.amazonaws.com"]
  thumbprint_list = ["6938fd4d98bab03faadb97b34396831e3780aea1"]
}

resource "aws_iam_role" "github_actions" {
  name = "github-actions-deploy"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Principal = {
        Federated = aws_iam_openid_connect_provider.github.arn
      }
      Action = "sts:AssumeRoleWithWebIdentity"
      Condition = {
        StringEquals = {
          "token.actions.githubusercontent.com:aud" = "sts.amazonaws.com"
        }
        StringLike = {
          "token.actions.githubusercontent.com:sub" = "repo:org/my-app:ref:refs/heads/main"
        }
      }
    }]
  })
}
```

### 3. ECS Fargate Service

```hcl
# ecs.tf
resource "aws_ecs_cluster" "main" {
  name = "production"

  setting {
    name  = "containerInsights"
    value = "enabled"
  }

  configuration {
    execute_command_configuration {
      logging = "OVERRIDE"
      log_configuration {
        cloud_watch_log_group_name = aws_cloudwatch_log_group.ecs.name
      }
    }
  }
}

resource "aws_ecs_task_definition" "app" {
  family                   = "my-app"
  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  cpu                      = 512
  memory                   = 1024
  execution_role_arn       = aws_iam_role.ecs_execution.arn
  task_role_arn            = aws_iam_role.app_task.arn

  container_definitions = jsonencode([{
    name      = "app"
    image     = "${aws_ecr_repository.app.repository_url}:${var.image_tag}"
    essential = true

    portMappings = [{
      containerPort = 8080
      protocol      = "tcp"
    }]

    environment = [
      { name = "NODE_ENV", value = "production" },
      { name = "AWS_REGION", value = var.aws_region }
    ]

    secrets = [
      { name = "DB_PASSWORD", valueFrom = aws_secretsmanager_secret.db_password.arn }
    ]

    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"         = aws_cloudwatch_log_group.app.name
        "awslogs-region"        = var.aws_region
        "awslogs-stream-prefix" = "app"
      }
    }

    healthCheck = {
      command     = ["CMD-SHELL", "curl -f http://localhost:8080/health || exit 1"]
      interval    = 30
      timeout     = 5
      retries     = 3
      startPeriod = 15
    }
  }])
}

resource "aws_ecs_service" "app" {
  name            = "my-app"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = 3
  launch_type     = "FARGATE"

  deployment_configuration {
    maximum_percent         = 200
    minimum_healthy_percent = 100
  }

  network_configuration {
    subnets          = module.vpc.private_subnets
    security_groups  = [aws_security_group.app.id]
    assign_public_ip = false
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.app.arn
    container_name   = "app"
    container_port   = 8080
  }

  deployment_circuit_breaker {
    enable   = true
    rollback = true
  }
}
```

### 4. RDS with Multi-AZ and Automated Backups

```hcl
# rds.tf
resource "aws_db_subnet_group" "main" {
  name       = "production"
  subnet_ids = module.vpc.intra_subnets
}

resource "aws_rds_cluster" "main" {
  cluster_identifier     = "production-db"
  engine                 = "aurora-postgresql"
  engine_version         = "15.4"
  database_name          = "myapp"
  master_username        = "dbadmin"
  manage_master_user_password = true  # Auto-generated, stored in Secrets Manager

  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.db.id]

  storage_encrypted = true
  kms_key_id        = aws_kms_key.rds.arn

  backup_retention_period = 30
  preferred_backup_window = "03:00-04:00"
  skip_final_snapshot     = false
  final_snapshot_identifier = "production-final-snapshot"
  deletion_protection     = true

  serverlessv2_scaling_configuration {
    min_capacity = 0.5
    max_capacity = 16
  }
}

resource "aws_rds_cluster_instance" "main" {
  count              = 2
  identifier         = "production-db-${count.index}"
  cluster_identifier = aws_rds_cluster.main.id
  instance_class     = "db.serverless"
  engine             = aws_rds_cluster.main.engine
  engine_version     = aws_rds_cluster.main.engine_version
}
```

### 5. S3 Bucket with Security and Lifecycle

```hcl
# s3.tf
resource "aws_s3_bucket" "app_data" {
  bucket = "my-app-production-data"
}

resource "aws_s3_bucket_versioning" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "app_data" {
  bucket = aws_s3_bucket.app_data.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.s3.arn
    }
    bucket_key_enabled = true
  }
}

resource "aws_s3_bucket_public_access_block" "app_data" {
  bucket                  = aws_s3_bucket.app_data.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_lifecycle_configuration" "app_data" {
  bucket = aws_s3_bucket.app_data.id

  rule {
    id     = "archive-old-uploads"
    status = "Enabled"

    filter {
      prefix = "uploads/"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }
}
```

### 6. Cost Optimization Tags

```hcl
# cost-tags.tf
locals {
  cost_tags = {
    Environment = var.environment
    Project     = "my-app"
    Team        = "platform"
    CostCenter  = "eng-${var.environment}"
    ManagedBy   = "terraform"
  }
}

# AWS Budget alert
resource "aws_budgets_budget" "monthly" {
  name              = "my-app-${var.environment}-monthly"
  budget_type       = "COST"
  limit_amount      = "5000"
  limit_unit        = "USD"
  time_unit         = "MONTHLY"
  time_period_start = "2026-01-01_00:00"

  cost_filter {
    name   = "TagKeyValue"
    values = ["user:Project$my-app"]
  }

  notification {
    comparison_operator        = "GREATER_THAN"
    threshold                  = 80
    threshold_type             = "PERCENTAGE"
    notification_type          = "FORECASTED"
    subscriber_email_addresses = ["platform@example.com"]
  }
}
```

## Best Practices / 最佳实践

1. **Use IAM roles, never access keys** — ECS task roles and EC2 instance profiles eliminate static credentials. Use OIDC for CI/CD.
2. **Enable VPC flow logs** — audit all network traffic. Essential for security investigations and compliance.
3. **Use SSM Parameter Store for config** — cheaper than Secrets Manager for non-secret configuration values.
4. **Enable CloudTrail in all regions** — multi-region trail with S3 bucket in a separate security account.
5. **Use AWS Organizations with SCPs** — prevent actions across accounts (e.g., no CloudTrail disabling).
6. **Right-size instances** — use AWS Compute Optimizer recommendations. Review monthly.
7. **Use Savings Plans** — for predictable workloads, 1-year compute savings plans offer 40-60% discount.
8. **Enable S3 Block Public Access** — account-level setting. Override only when explicitly needed.
9. **Use VPC endpoints** — S3, DynamoDB, ECR endpoints reduce NAT gateway costs and improve latency.
10. **Tag everything** — consistent tags for cost allocation, automation, and access control. Enforce with tag policies.

## Pitfalls / 常见陷阱

1. **Overly permissive IAM** — `*` in Action or Resource is a security hole. Use IAM Access Analyzer to identify unused permissions.
2. **Public S3 buckets** — even with good intentions, misconfigured bucket policies expose data. Use Block Public Access at the account level.
3. **NAT gateway costs** — NAT gateways charge per GB. Use VPC endpoints for AWS services to avoid NAT costs.
4. **Single NAT gateway** — a single NAT gateway is a single point of failure and bandwidth bottleneck. Use one per AZ for HA.
5. **RDS without encryption** — once created without encryption, you can't add it later. Always enable encryption at creation time.
6. **Missing deletion protection** — production databases and S3 buckets should have deletion protection enabled.
7. **Stale AMIs and snapshots** — old EBS snapshots and AMIs accumulate costs. Implement lifecycle policies.
8. **Cross-AZ data transfer** — traffic between AZs costs $0.01/GB. Co-locate chatty services in the same AZ.
9. **CloudWatch log retention** — logs default to "never expire." Set retention to 30-90 days for non-critical logs.
10. **Hardcoded account IDs** — use `data.aws_caller_identity` in Terraform instead of hardcoding account IDs.
