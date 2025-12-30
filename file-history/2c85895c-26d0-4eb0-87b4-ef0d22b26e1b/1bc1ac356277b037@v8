# AWS Deployment Plan for AstroTask (ECS Fargate)

**Selected Options:**
- Architecture: ECS Fargate
- Domain: Single domain (astrotask.io with /api for backend)
- AWS Account: Ready

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Namecheap DNS                                │
│               astrotask.io → ALB DNS name                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              Application Load Balancer (ALB)                    │
│         ├─ HTTPS :443 (ACM certificate - FREE)                  │
│         └─ HTTP :80 → Redirect to HTTPS                         │
└─────────────────────────────────────────────────────────────────┘
                     │                    │
              /api/* routes         All other routes
                     │                    │
                     ▼                    ▼
┌──────────────────────────┐  ┌──────────────────────────┐
│    ECS Fargate Service   │  │    ECS Fargate Service   │
│       (Backend)          │  │       (Frontend)         │
│      Port 8080           │  │       Port 3000          │
│    512 CPU / 1GB RAM     │  │    256 CPU / 512MB RAM   │
└──────────────────────────┘  └──────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    RDS PostgreSQL 16                            │
│              db.t3.micro (Private subnet)                       │
└─────────────────────────────────────────────────────────────────┘
```

**Estimated Cost: ~$60-80/month**

---

## Pre-Deployment: Revoke Compromised Secrets

**Do this FIRST before anything else:**

1. Go to https://resend.com/api-keys and revoke the exposed key
2. Generate new secrets locally:
   ```bash
   # JWT secret (64+ characters)
   openssl rand -base64 48

   # Database password (alphanumeric, no special chars for URL safety)
   openssl rand -base64 24 | tr -dc 'a-zA-Z0-9'
   ```
3. Create a new Resend API key at resend.com

---

## Files to Create

| File | Purpose |
|------|---------|
| `deploy/ecs/backend-task-definition.json` | ECS task config for backend |
| `deploy/ecs/frontend-task-definition.json` | ECS task config for frontend |
| `deploy/ecs/deploy.sh` | Deployment script |
| `.env.example` | Template showing required env vars |

---

## Step-by-Step Deployment

### Phase 1: AWS CLI Setup (~5 min)

```bash
# Install AWS CLI if needed
brew install awscli

# Configure with your credentials
aws configure
# Enter: Access Key ID, Secret Access Key, Region (us-east-1), Output (json)

# Verify it works
aws sts get-caller-identity
```

### Phase 2: Create ECR Repositories (~5 min)

```bash
# Set your AWS account ID (find in AWS Console top-right)
export AWS_ACCOUNT_ID=123456789012
export AWS_REGION=us-east-1

# Create repos
aws ecr create-repository --repository-name astrotask-backend --region $AWS_REGION
aws ecr create-repository --repository-name astrotask-frontend --region $AWS_REGION
```

### Phase 3: Create RDS Database (~15 min)

1. Go to **RDS Console** → **Create database**
2. Configuration:
   - Engine: **PostgreSQL 16**
   - Template: **Free tier** (for now, upgrade later)
   - DB instance identifier: `astrotask-db`
   - Master username: `postgres`
   - Master password: **Your new generated password**
   - Instance: `db.t3.micro`
   - Storage: 20 GB gp3
   - VPC: **Default VPC**
   - Public access: **No**
   - VPC security group: Create new → `astrotask-db-sg`
   - Database name: `astrotask_db`

3. **Note the Endpoint** (e.g., `astrotask-db.xxxxx.us-east-1.rds.amazonaws.com`)

### Phase 4: Store Secrets in AWS Secrets Manager (~5 min)

```bash
aws secretsmanager create-secret \
  --name astrotask/production \
  --description "AstroTask production secrets" \
  --secret-string '{
    "DATABASE_URL": "postgresql://postgres:YOUR_PASSWORD@YOUR_RDS_ENDPOINT:5432/astrotask_db?sslmode=require",
    "JWT_SECRET": "YOUR_64_CHAR_SECRET",
    "RESEND_API_KEY": "re_YOUR_NEW_KEY",
    "POSTGRES_PASSWORD": "YOUR_PASSWORD",
    "FRONTEND_URL": "https://astrotask.io",
    "CORS_ORIGINS": "https://astrotask.io",
    "EMAIL_FROM": "noreply@astrotask.io",
    "ADMIN_EMAIL": "your-email@example.com",
    "ENVIRONMENT": "production",
    "SECURE_COOKIES": "true",
    "REQUIRE_INVITE": "true"
  }'
```

### Phase 5: Request SSL Certificate (~5 min + validation time)

1. Go to **ACM (Certificate Manager)** → **Request certificate**
2. Request a **public certificate**
3. Domain names:
   - `astrotask.io`
   - `*.astrotask.io` (wildcard for future subdomains)
4. Validation method: **DNS validation**
5. Click **Request**
6. On the certificate page, click **Create records in Route 53** OR copy the CNAME values

**For Namecheap DNS:**
1. Log into Namecheap → Domain List → Manage → Advanced DNS
2. Add CNAME record:
   - Host: The `_xxxxx` part from ACM (without domain)
   - Value: The ACM validation value
3. Wait ~5-30 minutes for validation (status changes to "Issued")

### Phase 6: Create ECS Cluster (~5 min)

1. Go to **ECS Console** → **Create cluster**
2. Configuration:
   - Cluster name: `astrotask-cluster`
   - Infrastructure: **AWS Fargate (serverless)**
3. Click **Create**

### Phase 7: Create Application Load Balancer (~10 min)

1. Go to **EC2 Console** → **Load Balancers** → **Create Load Balancer**
2. Select **Application Load Balancer**
3. Basic configuration:
   - Name: `astrotask-alb`
   - Scheme: **Internet-facing**
   - IP type: IPv4
4. Network mapping:
   - VPC: Default VPC
   - Subnets: Select at least 2 availability zones
5. Security groups:
   - Create new: `astrotask-alb-sg`
   - Inbound rules: HTTP (80) and HTTPS (443) from anywhere
6. Listeners:
   - HTTP:80 → Redirect to HTTPS:443
   - HTTPS:443 → Forward to target group (create next)
7. Select your ACM certificate for HTTPS

**Create Target Groups:**
1. `astrotask-backend-tg`:
   - Target type: IP
   - Port: 8080
   - Health check: `/health`
   - Protocol: HTTP
2. `astrotask-frontend-tg`:
   - Target type: IP
   - Port: 3000
   - Health check: `/`
   - Protocol: HTTP

**Add Routing Rules (after ALB created):**
1. Go to ALB → Listeners → HTTPS:443 → View/edit rules
2. Add rule:
   - IF path is `/api/*` → Forward to `astrotask-backend-tg`
3. Default action: Forward to `astrotask-frontend-tg`

### Phase 8: Build and Push Docker Images (~15 min)

```bash
# Login to ECR
aws ecr get-login-password --region $AWS_REGION | \
  docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

# Build and push backend
cd backend
docker build -t astrotask-backend .
docker tag astrotask-backend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/astrotask-backend:latest
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/astrotask-backend:latest

# Build and push frontend (with production API URL)
cd ..
docker build \
  --build-arg NEXT_PUBLIC_API_URL=https://astrotask.io/api/v1 \
  -f frontend/Dockerfile \
  -t astrotask-frontend .
docker tag astrotask-frontend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/astrotask-frontend:latest
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/astrotask-frontend:latest
```

### Phase 9: Create ECS Task Definitions (~10 min)

**Backend Task Definition** (register via CLI or Console):
```json
{
  "family": "astrotask-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [{
    "name": "backend",
    "image": "ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/astrotask-backend:latest",
    "portMappings": [{"containerPort": 8080, "protocol": "tcp"}],
    "secrets": [
      {"name": "DATABASE_URL", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:DATABASE_URL::"},
      {"name": "JWT_SECRET", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:JWT_SECRET::"},
      {"name": "RESEND_API_KEY", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:RESEND_API_KEY::"},
      {"name": "FRONTEND_URL", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:FRONTEND_URL::"},
      {"name": "CORS_ORIGINS", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:CORS_ORIGINS::"},
      {"name": "EMAIL_FROM", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:EMAIL_FROM::"},
      {"name": "ADMIN_EMAIL", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:ADMIN_EMAIL::"},
      {"name": "ENVIRONMENT", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:ENVIRONMENT::"},
      {"name": "SECURE_COOKIES", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:SECURE_COOKIES::"},
      {"name": "REQUIRE_INVITE", "valueFrom": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/production:REQUIRE_INVITE::"}
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/astrotask-backend",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}
```

**Frontend Task Definition:**
```json
{
  "family": "astrotask-frontend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "containerDefinitions": [{
    "name": "frontend",
    "image": "ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/astrotask-frontend:latest",
    "portMappings": [{"containerPort": 3000, "protocol": "tcp"}],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/astrotask-frontend",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }]
}
```

### Phase 10: Create ECS Services (~10 min)

1. Go to **ECS** → **astrotask-cluster** → **Create service**

**Backend Service:**
- Launch type: Fargate
- Task definition: astrotask-backend
- Service name: astrotask-backend-service
- Desired tasks: 1
- Networking:
  - VPC: Default
  - Subnets: Private subnets (or public with auto-assign public IP)
  - Security group: Create new, allow 8080 from ALB security group
- Load balancing:
  - Application Load Balancer
  - Select `astrotask-alb`
  - Target group: `astrotask-backend-tg`

**Frontend Service:**
- Launch type: Fargate
- Task definition: astrotask-frontend
- Service name: astrotask-frontend-service
- Desired tasks: 1
- Same networking as backend (allow 3000 from ALB)
- Load balancing:
  - Select `astrotask-alb`
  - Target group: `astrotask-frontend-tg`

### Phase 11: Configure RDS Security Group (~5 min)

1. Go to **EC2** → **Security Groups** → Find `astrotask-db-sg`
2. Edit inbound rules:
   - Type: PostgreSQL (5432)
   - Source: The security group of your ECS tasks

### Phase 12: Configure DNS on Namecheap (~5 min)

1. Go to Namecheap → Domain List → Manage → Advanced DNS
2. Delete any existing A records for @ and www
3. Add CNAME record:
   - Host: `www`
   - Value: ALB DNS name (e.g., `astrotask-alb-123456.us-east-1.elb.amazonaws.com`)

**Note:** Namecheap doesn't support ALIAS records for apex domain (@). Options:
- Use `www.astrotask.io` as primary, redirect apex to www via Namecheap
- Or switch to Route 53 for DNS ($0.50/month per zone)

4. Wait 5-30 minutes for propagation

### Phase 13: Verify Deployment (~5 min)

```bash
# Check ALB health
curl -I https://www.astrotask.io

# Check API health
curl https://www.astrotask.io/api/v1/health

# View backend logs
aws logs tail /ecs/astrotask-backend --follow

# View frontend logs
aws logs tail /ecs/astrotask-frontend --follow
```

---

## Security Group Summary

| Security Group | Inbound Rules |
|----------------|---------------|
| `astrotask-alb-sg` | HTTP 80 (0.0.0.0/0), HTTPS 443 (0.0.0.0/0) |
| `astrotask-ecs-sg` | 8080 (astrotask-alb-sg), 3000 (astrotask-alb-sg) |
| `astrotask-db-sg` | 5432 (astrotask-ecs-sg) |

---

## IAM Role Required

ECS tasks need `ecsTaskExecutionRole` with policies:
- `AmazonECSTaskExecutionRolePolicy` (built-in)
- Custom policy for Secrets Manager access:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": "arn:aws:secretsmanager:us-east-1:ACCOUNT_ID:secret:astrotask/*"
  }]
}
```

---

## Post-Deployment

1. **First Login:**
   - Check backend logs for bootstrap admin invite URL
   - Use that URL to create your admin account

2. **Monitoring:**
   - Set up CloudWatch alarms for ECS task health
   - Enable RDS Performance Insights

3. **Future Deployments:**
   ```bash
   # Rebuild and push new image
   docker build ... && docker push ...

   # Force new deployment
   aws ecs update-service --cluster astrotask-cluster --service astrotask-backend-service --force-new-deployment
   ```

---

## Deployment Checklist

- [ ] Revoke compromised Resend API key
- [ ] Generate new JWT_SECRET and POSTGRES_PASSWORD
- [ ] Create ECR repositories
- [ ] Create RDS database
- [ ] Store secrets in Secrets Manager
- [ ] Request and validate ACM certificate
- [ ] Create ECS cluster
- [ ] Create ALB with target groups and routing rules
- [ ] Build and push Docker images
- [ ] Create ECS task definitions
- [ ] Create ECS services
- [ ] Configure security groups
- [ ] Update DNS on Namecheap
- [ ] Verify deployment works
- [ ] Create admin account via invite link
