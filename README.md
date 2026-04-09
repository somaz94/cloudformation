# CloudFormation Templates

A collection of reusable AWS CloudFormation templates organized by service category.

<br/>

## Structure

```
cloudformation/
├── networking/          # VPC, CDN, DNS
│   ├── vpc.yml          # VPC + Subnets + NAT Gateway + Security Groups
│   ├── cloudfront.yml   # CloudFront Distribution + S3 Origin + OAC
│   └── route53.yml      # Route53 Hosted Zone + DNS Records
├── compute/             # Servers, Containers, Serverless
│   ├── ecs.yml          # ECS Fargate + ALB + Auto Scaling
│   ├── ec2.yml          # EC2 + ASG + ALB + Launch Template
│   ├── lambda.yml       # Lambda + API Gateway REST API
│   └── eks.yml          # EKS Cluster + Managed Node Group + OIDC
├── database/            # Relational, NoSQL, Cache
│   ├── rds.yml          # Aurora PostgreSQL + Auto Scaling + Secrets Manager
│   ├── dynamodb.yml     # DynamoDB Table + GSI + Point-in-Time Recovery
│   └── elasticache.yml  # ElastiCache Redis + Replication Group
├── storage/             # Object Storage, File Systems
│   ├── s3.yml           # S3 Bucket + Bucket Policy
│   └── efs.yml          # EFS FileSystem + Mount Targets
├── security/            # IAM, WAF
│   ├── iam.yml          # IAM Roles (ECS, Lambda, GitHub Actions OIDC)
│   └── waf.yml          # WAFv2 WebACL + Managed Rules + Rate Limiting
├── messaging/           # Queues, Notifications
│   ├── sqs.yml          # SQS Queue + Dead Letter Queue
│   └── sns.yml          # SNS Topic + Subscriptions
├── monitoring/          # Observability
│   └── cloudwatch.yml   # CloudWatch Alarms + Dashboard
└── cicd/                # CI/CD Pipeline
    └── codepipeline.yml # CodePipeline + CodeBuild + ECR
```

<br/>

## Conventions

All templates follow a consistent pattern:

| Convention | Description |
|------------|-------------|
| **Parameters** | `ProjectName` (String) + `ENV` (dev/prod) |
| **Naming** | `${ProjectName}-${ENV}-{resource}` |
| **Conditions** | `IsProd` for production-only features (autoscaling, HA, etc.) |
| **Exports** | Cross-stack references via `${ProjectName}-${ENV}-{ResourceType}` |
| **Tags** | `Project` + `Environment` on all taggable resources |

<br/>

## Deployment Order

Templates have cross-stack dependencies. Deploy in this order:

```
1. networking/vpc.yml        (foundation)
2. security/iam.yml          (IAM roles)
3. database/rds.yml          (depends on VPC)
   database/dynamodb.yml     (independent)
   database/elasticache.yml  (depends on VPC)
   storage/s3.yml            (independent)
   storage/efs.yml           (depends on VPC)
   messaging/sqs.yml         (independent)
   messaging/sns.yml         (independent)
4. compute/ecs.yml           (depends on VPC, IAM)
   compute/ec2.yml           (depends on VPC)
   compute/lambda.yml        (depends on IAM)
   compute/eks.yml           (depends on VPC, IAM)
5. networking/cloudfront.yml (depends on S3)
   networking/route53.yml    (depends on CloudFront)
   security/waf.yml          (depends on ALB)
6. monitoring/cloudwatch.yml (depends on ECS, RDS)
   cicd/codepipeline.yml     (depends on ECS)
```

<br/>

## Usage

### Deploy a stack

```bash
aws cloudformation deploy \
  --template-file networking/vpc.yml \
  --stack-name myapp-dev-vpc \
  --parameter-overrides ProjectName=myapp ENV=dev \
  --capabilities CAPABILITY_NAMED_IAM
```

### Validate a template

```bash
aws cloudformation validate-template \
  --template-body file://networking/vpc.yml
```

### Delete a stack

```bash
aws cloudformation delete-stack \
  --stack-name myapp-dev-vpc
```

<br/>

## Template Summary

| Category | Template | Key Resources |
|----------|----------|---------------|
| Networking | vpc.yml | VPC, 2 Public + 2 Private Subnets, NAT Gateway, IGW |
| Networking | cloudfront.yml | CloudFront Distribution, Origin Access Control |
| Networking | route53.yml | Hosted Zone, A/CNAME/MX Records |
| Compute | ecs.yml | ECS Fargate, ALB, Task Definitions, Auto Scaling |
| Compute | ec2.yml | Launch Template, ASG, ALB, Target Group |
| Compute | lambda.yml | Lambda Function, API Gateway, IAM Role |
| Compute | eks.yml | EKS Cluster, Managed Node Group, OIDC Provider |
| Database | rds.yml | Aurora PostgreSQL, Secrets Manager, Read Replica Scaling |
| Database | dynamodb.yml | DynamoDB Table, GSI, PITR |
| Database | elasticache.yml | ElastiCache Redis, Replication Group |
| Storage | s3.yml | S3 Bucket, Public Access Policy |
| Storage | efs.yml | EFS FileSystem, Mount Targets |
| Security | iam.yml | IAM Roles (ECS, Lambda, GitHub Actions OIDC) |
| Security | waf.yml | WAFv2 WebACL, AWS Managed Rules, Rate Limiting |
| Messaging | sqs.yml | SQS Queue, Dead Letter Queue |
| Messaging | sns.yml | SNS Topic, Email/SQS Subscriptions |
| Monitoring | cloudwatch.yml | CloudWatch Alarms, Dashboard |
| CI/CD | codepipeline.yml | CodePipeline, CodeBuild, ECR |
