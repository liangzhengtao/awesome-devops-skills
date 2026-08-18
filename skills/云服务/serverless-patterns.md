# Serverless Architecture Patterns / Serverless 架构模式

> Event-driven serverless patterns with AWS Lambda, API Gateway, Step Functions, EventBridge, and cost-optimized architectures.

## When to Use / 何时使用

Use this skill when:

- Building APIs with Lambda + API Gateway
- Designing event-driven architectures with EventBridge or SNS/SQS
- Implementing async workflows with Step Functions
- Processing S3 uploads, DynamoDB streams, or Kinesis events
- Optimizing Lambda cold starts and execution costs
- Building scheduled tasks and cron jobs with EventBridge Scheduler
- Migrating from always-on servers to serverless

## Architecture / 架构

```
┌──────────────────────────────────────────────────────────────────┐
│                    Serverless Architecture                         │
│                                                                   │
│  Client ──► API Gateway ──► Lambda (Authorizer)                  │
│                │                                                  │
│                ├──► Lambda (API Handler) ──► DynamoDB             │
│                │                                                  │
│                └──► Lambda (WebSocket) ──► API Gateway WS        │
│                                                                   │
│  S3 Upload ──► Event Notification ──► Lambda (Process)           │
│                                          │                        │
│                                          ├──► SQS (Queue)        │
│                                          └──► SNS (Notify)       │
│                                                                   │
│  EventBridge ──► Rule ──► Lambda (Scheduler)                     │
│                │                                                  │
│                └──► Step Functions ──► Lambda A ──► Lambda B      │
│                                              ──► Lambda C (parallel)│
│                                                                   │
│  Cost: $0 for idle, pay per invocation                           │
│  Scale: 0 → 1000+ concurrent in seconds                         │
└──────────────────────────────────────────────────────────────────┘
```

## Code Templates / 代码模板

### 1. Lambda Function with API Gateway (TypeScript)

```typescript
// src/handlers/users.ts
import { APIGatewayProxyEvent, APIGatewayProxyResult } from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, GetCommand, PutCommand, QueryCommand } from '@aws-sdk/lib-dynamodb';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client, {
  marshallOptions: { removeUndefinedValues: true },
});

const TABLE_NAME = process.env.TABLE_NAME!;

export const getUser = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  const userId = event.pathParameters?.id;
  if (!userId) {
    return { statusCode: 400, body: JSON.stringify({ error: 'Missing user ID' }) };
  }

  const result = await docClient.send(new GetCommand({
    TableName: TABLE_NAME,
    Key: { pk: `USER#${userId}`, sk: `PROFILE` },
  }));

  if (!result.Item) {
    return { statusCode: 404, body: JSON.stringify({ error: 'User not found' }) };
  }

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json', 'Cache-Control': 'max-age=60' },
    body: JSON.stringify(result.Item),
  };
};

export const createUser = async (event: APIGatewayProxyEvent): Promise<APIGatewayProxyResult> => {
  const body = JSON.parse(event.body || '{}');

  if (!body.email || !body.name) {
    return { statusCode: 400, body: JSON.stringify({ error: 'Missing required fields' }) };
  }

  const userId = crypto.randomUUID();
  const now = new Date().toISOString();

  await docClient.send(new PutCommand({
    TableName: TABLE_NAME,
    Item: {
      pk: `USER#${userId}`,
      sk: 'PROFILE',
      userId,
      email: body.email,
      name: body.name,
      createdAt: now,
      updatedAt: now,
    },
    ConditionExpression: 'attribute_not_exists(pk)',
  }));

  return {
    statusCode: 201,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ userId, email: body.email, name: body.name }),
  };
};
```

### 2. SAM/CloudFormation Template

```yaml
# template.yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Serverless API with DynamoDB

Globals:
  Function:
    Runtime: nodejs20.x
    Architectures: [arm64]        # Graviton2: 20% cheaper, 20% faster
    MemorySize: 256
    Timeout: 30
    Tracing: Active               # X-Ray tracing
    Environment:
      Variables:
        TABLE_NAME: !Ref UsersTable
        NODE_OPTIONS: '--enable-source-maps'
    Layers:
      - !Ref SharedLayer

Resources:
  # ── API Gateway ──────────────────────────────────
  Api:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod
      TracingEnabled: true
      Cors: "'*'"
      Auth:
        DefaultAuthorizer: CognitoAuthorizer
        Authorizers:
          CognitoAuthorizer:
            UserPoolArn: !GetAtt UserPool.Arn
      MethodSettings:
        - ResourcePath: '/*'
          HttpMethod: '*'
          ThrottlingBurstLimit: 100
          ThrottlingRateLimit: 50

  # ── Lambda Functions ─────────────────────────────
  GetUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${AWS::StackName}-get-user'
      Handler: dist/handlers/users.getUser
      Description: Get user by ID
      Events:
        GetUser:
          Type: Api
          Properties:
            RestApiId: !Ref Api
            Path: /users/{id}
            Method: get
      Policies:
        - DynamoDBReadPolicy:
            TableName: !Ref UsersTable

  CreateUserFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${AWS::StackName}-create-user'
      Handler: dist/handlers/users.createUser
      Description: Create new user
      ReservedConcurrentExecutions: 100
      Events:
        CreateUser:
          Type: Api
          Properties:
            RestApiId: !Ref Api
            Path: /users
            Method: post
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref UsersTable

  # ── Image Processing (S3 Event) ──────────────────
  ProcessImageFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${AWS::StackName}-process-image'
      Handler: dist/handlers/images.process
      MemorySize: 1024
      Timeout: 60
      Events:
        S3Event:
          Type: S3
          Properties:
            Bucket: !Ref UploadBucket
            Events: s3:ObjectCreated:*
            Filter:
              S3Key:
                Rules:
                  - Name: prefix
                    Value: uploads/
                  - Name: suffix
                    Value: .jpg

  # ── Scheduled Task ───────────────────────────────
  CleanupFunction:
    Type: AWS::Serverless::Function
    Properties:
      FunctionName: !Sub '${AWS::StackName}-cleanup'
      Handler: dist/handlers/cleanup.handler
      Description: Daily cleanup of expired data
      Events:
        Schedule:
          Type: Schedule
          Properties:
            Schedule: 'rate(1 day)'
            Enabled: true
            Input:
              '{"retentionDays": 90}'

  # ── DynamoDB ─────────────────────────────────────
  UsersTable:
    Type: AWS::DynamoDB::Table
    DeletionPolicy: Retain
    Properties:
      TableName: !Sub '${AWS::StackName}-users'
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - { AttributeName: pk, AttributeType: S }
        - { AttributeName: sk, AttributeType: S }
        - { AttributeName: gsi1pk, AttributeType: S }
        - { AttributeName: gsi1sk, AttributeType: S }
      KeySchema:
        - { AttributeName: pk, KeyType: HASH }
        - { AttributeName: sk, KeyType: RANGE }
      GlobalSecondaryIndexes:
        - IndexName: gsi1
          KeySchema:
            - { AttributeName: gsi1pk, KeyType: HASH }
            - { AttributeName: gsi1sk, KeyType: RANGE }
          Projection: { ProjectionType: ALL }
      PointInTimeRecoverySpecification:
        PointInTimeRecoveryEnabled: true
      SSESpecification:
        SSEEnabled: true

  # ── Shared Layer ─────────────────────────────────
  SharedLayer:
    Type: AWS::Serverless::LayerVersion
    Properties:
      LayerName: shared-dependencies
      ContentUri: layers/shared/
      CompatibleRuntimes: [nodejs20.x]
      CompatibleArchitectures: [arm64]

Outputs:
  ApiEndpoint:
    Value: !Sub 'https://${Api}.execute-api.${AWS::Region}.amazonaws.com/prod'
```

### 3. Step Functions Workflow

```yaml
# In template.yaml
OrderWorkflow:
  Type: AWS::Serverless::StateMachine
  Properties:
    Type: EXPRESS
    Logging:
      Level: ALL
      IncludeExecutionData: true
    DefinitionUri: workflows/order.asl.json
    DefinitionSubstitutions:
      ValidateFunction: !GetAtt ValidateOrderFunction.Arn
      ProcessFunction: !GetAtt ProcessOrderFunction.Arn
      NotifyFunction: !GetAtt NotifyCustomerFunction.Arn
```

```json
// workflows/order.asl.json
{
  "Comment": "Order processing workflow",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "${ValidateFunction}",
      "Next": "ProcessOrder",
      "Retry": [
        {
          "ErrorEquals": ["States.TaskFailed"],
          "IntervalSeconds": 2,
          "MaxAttempts": 3,
          "BackoffRate": 2
        }
      ],
      "Catch": [
        {
          "ErrorEquals": ["ValidationError"],
          "Next": "HandleInvalidOrder"
        }
      ]
    },
    "ProcessOrder": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "ChargePayment",
          "States": {
            "ChargePayment": {
              "Type": "Task",
              "Resource": "arn:aws:states:::lambda:invoke",
              "Parameters": {
                "FunctionName": "${ProcessFunction}",
                "Payload": { "action": "charge", "order.$": "$" }
              },
              "End": true
            }
          }
        },
        {
          "StartAt": "ReserveInventory",
          "States": {
            "ReserveInventory": {
              "Type": "Task",
              "Resource": "arn:aws:states:::dynamodb:putItem",
              "Parameters": {
                "TableName": "inventory",
                "Item": {
                  "PK": { "S.$": "States.Format('ORDER#{}', $.orderId)" },
                  "status": { "S": "reserved" }
                }
              },
              "End": true
            }
          }
        }
      ],
      "Next": "NotifyCustomer"
    },
    "NotifyCustomer": {
      "Type": "Task",
      "Resource": "${NotifyFunction}",
      "End": true
    },
    "HandleInvalidOrder": {
      "Type": "Task",
      "Resource": "${NotifyFunction}",
      "Parameters": {
        "message": "Order validation failed",
        "order.$": "$"
      },
      "End": true
    }
  }
}
```

### 4. EventBridge Rules

```yaml
# In template.yaml
  # React to S3 events
  S3EventRule:
    Type: AWS::Events::Rule
    Properties:
      Description: Forward S3 events to processing
      EventPattern:
        source: ["aws.s3"]
        detail-type: ["Object Created"]
        detail:
          bucket:
            name: [!Ref UploadBucket]
          object:
            key:
              - prefix: "uploads/"
      Targets:
        - Arn: !GetAtt ProcessImageFunction.Arn
          Id: ProcessImage

  # Custom application events
  UserEventRule:
    Type: AWS::Events::Rule
    Properties:
      EventPattern:
        source: ["myapp.users"]
        detail-type: ["User Created", "User Updated"]
      Targets:
        - Arn: !GetAtt SyncUserFunction.Arn
          Id: SyncUser
        - Arn: !GetAtt WelcomeEmailFunction.Arn
          Id: WelcomeEmail
```

### 5. Cold Start Optimization

```typescript
// ── Module-level initialization (outside handler) ──
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';

// These execute once per cold start, not per invocation
const client = new DynamoDBClient({
  region: process.env.AWS_REGION,
  maxAttempts: 3,
});

// Connection reuse via module scope
const CONNECTION_POOL = new Map<string, any>();

// ── Handler with warm start benefit ──
export const handler = async (event: any) => {
  // Use the pre-initialized client
  // ...
};

// ── Provisioned concurrency config (SAM) ──
// Prevents cold starts entirely for critical paths
// ProvisionedConcurrency: 10
```

## Best Practices / 最佳实践

1. **Use ARM64 (Graviton2)** — 20% cheaper and better performance. Set `Architectures: [arm64]` in SAM.
2. **Initialize outside the handler** — database clients, HTTP connections, and SDK clients should be module-level for connection reuse.
3. **Use provisioned concurrency for critical paths** — eliminates cold starts for latency-sensitive APIs. Costs more but provides consistent performance.
4. **Right-size memory** — Lambda allocates CPU proportionally to memory. Use AWS Lambda Power Tuning to find the sweet spot.
5. **Use SQS for async processing** — decouple long-running work from API responses. SQS handles retries and dead-letter queues.
6. **Implement idempotency** — Lambda may invoke handlers more than once. Use DynamoDB idempotency keys for payment operations.
7. **Use DynamoDB single-table design** — one table for all entity types. Reduces round trips and simplifies access patterns.
8. **Enable X-Ray tracing** — `Tracing: Active` gives end-to-end visibility across Lambda, API Gateway, DynamoDB, and SQS.
9. **Set reserved concurrency** — prevents one function from consuming all account concurrency. Protects other functions.
10. **Use Layers for shared code** — extract common utilities, SDK clients, and dependencies into Lambda Layers.

## Pitfalls / 常见陷阱

1. **Cold start latency** — Node.js cold starts are 200-500ms; Java/Python can be 1-5s. Use provisioned concurrency or keep-alive pings.
2. **No connection pooling** — creating a new DB connection per invocation is slow and exhausting. Reuse connections at module scope.
3. **Timeout cascade** — Lambda timeout (30s) + API Gateway timeout (30s) + downstream timeout. Total must not exceed the outermost timeout.
4. **Memory leaks in warm containers** — module-level state persists across invocations. Don't accumulate data without cleanup.
5. **DynamoDB throttling** — on-demand mode handles bursts but has limits. Use provisioned capacity with auto-scaling for predictable loads.
6. **API Gateway payload limit** — 10MB for REST API, 128KB for WebSocket. Use presigned S3 URLs for large payloads.
7. **Step Functions cost** — at $0.025 per 1000 state transitions, complex workflows can be expensive. Batch operations where possible.
8. **Lambda concurrency limits** — default 1000 concurrent executions per region. Request a limit increase for high-traffic APIs.
9. **Dead letter queue monitoring** — DLQ messages are silently stored. Set up CloudWatch alarms on DLQ depth.
10. **Vendor lock-in** — serverless architectures are deeply tied to cloud provider services. Abstract critical business logic from provider-specific APIs.

---

## 中文版本

### 使用场景

- 使用 Lambda + API Gateway 构建 API
- 使用 EventBridge 或 SNS/SQS 设计事件驱动架构
- 使用 Step Functions 实现异步工作流
- 处理 S3 上传、DynamoDB 流或 Kinesis 事件
- 优化 Lambda 冷启动和执行成本
- 从常驻服务器迁移到 serverless

### 核心步骤

1. **Lambda + API Gateway** — TypeScript handler 实现 CRUD，使用 DynamoDB single-table design
2. **SAM/CloudFormation 模板** — 使用 `AWS::Serverless` 资源定义函数、API、DynamoDB，配置 arm64 架构节省成本
3. **Step Functions 工作流** — 定义顺序/并行/错误处理的状态机，支持 retry 和 catch
4. **EventBridge 规则** — 基于事件模式路由 S3 事件和自定义应用事件到对应 Lambda
5. **冷启动优化** — 模块级初始化（数据库客户端复用）、Provisioned Concurrency、arm64 架构

### 模板说明

- Lambda handler — 完整的 TypeScript CRUD handler，包含输入验证和错误处理
- SAM 模板 — API Gateway + Lambda + DynamoDB + S3 事件 + 定时任务的完整模板
- Step Functions — 订单处理工作流（验证 → 并行处理支付和库存 → 通知）
- EventBridge — S3 事件和自定义事件的规则配置

### 常见陷阱

1. **冷启动延迟** — Node.js 200-500ms，Java/Python 1-5s，关键路径使用 Provisioned Concurrency
2. **无连接复用** — 每次调用创建新 DB 连接又慢又耗资源，在模块级复用连接
3. **超时级联** — Lambda 超时(30s) + API Gateway 超时(30s) + 下游超时，总时间不能超过最外层
4. **DynamoDB 限流** — on-demand 模式处理突发但有限制，可预测负载使用 provisioned capacity + auto-scaling
5. **API Gateway payload 限制** — REST API 10MB，WebSocket 128KB，大文件使用 presigned S3 URL
