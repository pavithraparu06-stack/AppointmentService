# Hospital Management System - Production-Ready Microservices Architecture

Complete CI/CD infrastructure with 4 repositories, automated deployments, and AWS ECS Fargate integration.

---

## 📦 Project Overview

This is a complete, production-ready implementation of a hospital management system consisting of:

| Component | Technology | Location | Status |
|-----------|-----------|----------|--------|
| **Patient Microservice** | Node.js + Express + MySQL | `./PatientService` | ✅ Complete |
| **Appointment Microservice** | Node.js + Express + MySQL | `./AppointmentService` | ✅ Complete |
| **Patient Portal (Web UI)** | React + Vite + Nginx | `./patient-portal` | ✅ Complete |
| **Infrastructure-as-Code** | Terraform + AWS | `./Infrastructure` | ✅ Complete |

**Architecture:** 3 microservices deployed on AWS ECS Fargate with MySQL RDS, Application Load Balancer, and complete CI/CD pipelines.

---

## 🚀 Quick Start

### 1. Local Development

```bash
# Terminal 1: Patient Service (port 3001)
cd PatientService
npm install
npm run dev

# Terminal 2: Appointment Service (port 3002)
cd AppointmentService
npm install
npm run dev

# Terminal 3: Patient Portal (port 5173)
cd patient-portal
npm install
npm run dev

# Visit: http://localhost:5173
```

### 2. Git & GitHub

```bash
# Create 4 repositories on GitHub
# Then push code:

cd PatientService
git remote add origin https://github.com/YOUR_ORG/hospital-patient-service.git
git branch -M main && git push -u origin main

cd ../AppointmentService
git remote add origin https://github.com/YOUR_ORG/hospital-appointment-service.git
git branch -M main && git push -u origin main

cd ../patient-portal
git remote add origin https://github.com/YOUR_ORG/hospital-patient-portal.git
git branch -M main && git push -u origin main

cd ../Infrastructure
git remote add origin https://github.com/YOUR_ORG/hospital-infrastructure.git
git branch -M main && git push -u origin main
```

## 🔧 Required Configuration

Before deploying, ensure the following values are created and available (GitHub secrets or Terraform variables):

| Name | Purpose | Example / Notes |
|------|---------|-----------------|
| `AWS_ACCOUNT_ID` | AWS account identifier | 147997138755 |
| `AWS_REGION` | AWS region for deployment | us-east-1 |
| `TERRAFORM_STATE_BUCKET` | S3 bucket for Terraform state | hospital-management-tf-state |
| `TERRAFORM_DDB_TABLE` | DynamoDB table for state locks | terraform-locks |
| `AWS_ROLE_ARN` | (Optional) OIDC role for GitHub Actions | arn:aws:iam::147997138755:role/GithubActionsRole |
| `GHCR_REGISTRY` | (Optional) Container registry (GHCR) | ghcr.io/YOUR_ORG |
| `DB_NAME` | RDS database name | hospital_patient_db |
| `DB_USER` | RDS master username | admin |
| `DB_PASSWORD` | RDS master password | store in Secrets Manager / GitHub secret |
| `JWT_SECRET` | Application JWT secret | store in Secrets Manager |
| `EXISTING_VPC_ID` | (Optional) Use existing VPC | vpc-04c43f1e3f46e50f5 |
| `PUBLIC_SUBNET_IDS` | (Optional) Public subnets (comma-separated) | subnet-0ee8dd7c02b75b349,subnet-0bf538da9d3f0040c,subnet-04f718ecdfe67f77a |
| `PRIVATE_SUBNET_IDS` | (Optional) Private subnets (comma-separated) | subnet-0af4ff257380caf3f,subnet-0e04c326304dad454,subnet-07c3eadb68cb75f7f |

Note: If you supply `EXISTING_VPC_ID` and the subnet IDs, the infrastructure will use them; otherwise Terraform will create a new VPC and subnets.

### 3. Configure AWS

```bash
# Create S3 + DynamoDB for Terraform state
aws s3api create-bucket --bucket hospital-management-tf-state --region us-east-1
aws dynamodb create-table --table-name terraform-locks \
  --attribute-definitions AttributeName=LockID,AttributeType=S \
  --key-schema AttributeName=LockID,KeyType=HASH \
  --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5

# Create GitHub Actions IAM role (OIDC)
# Configure IAM OIDC role for CI and attach least-privilege AWS policies
```

### 4. Deploy Infrastructure

```bash
# Push Infrastructure repo to trigger CD pipeline
cd Infrastructure
git push origin main

# Monitor CI pipeline output
# Review Terraform plan, approve manual apply
```

---

## 🧰 Required Tools (Local + CI/CD)

| Tool | Required For | Install Notes |
|------|--------------|---------------|
| **Node.js 18+** | Build/run services + portal | `node` + `npm` |
| **Docker** | Build container images | Required on CI agents |
| **Terraform 1.6+** | IaC provisioning | Used by Infrastructure pipeline |
| **AWS CLI v2** | AWS access + ECS Exec | Required for ECS Exec + S3/DDB setup |
| **Session Manager Plugin** | ECS Exec sessions | Install on admin workstation |
| **GitHub Actions** | CI/CD orchestration | Workflows in each repo |
| **Trivy** | Image security scanning | Optional (soft-fail) |
| **Checkov** | IaC scanning | Optional (soft-fail) |
| **SonarQube Scanner** | Code quality scan | Optional |

---

## 🗄️ Database Migrations & Seeding

### Migrations (required once per environment)

Run **after** infrastructure is applied and the RDS endpoint is available.

**Via ECS Exec (recommended):**
```bash
aws ecs execute-command \
  --cluster hospital-management-prod-cluster \
  --task <TASK_ID> \
  --container patient-service \
  --interactive \
  --command "npm run db:migrate"
```

**Alternative:** run a one-off ECS task with command `npm run db:migrate`.

### Seed Data (optional)

Adds demo patients for the portal list.
```bash
aws ecs execute-command \
  --cluster hospital-management-prod-cluster \
  --task <TASK_ID> \
  --container patient-service \
  --interactive \
  --command "npm run db:seed"
```

### Reset (destructive)
```bash
aws ecs execute-command \
  --cluster hospital-management-prod-cluster \
  --task <TASK_ID> \
  --container patient-service \
  --interactive \
  --command "npm run db:seed:undo"
```

---

## 📋 Directory Structure

```
Patient/
│
├── PatientService/                    # Microservice #1
│   ├── .github/workflows/
│   │   ├── ci-build.yml               # Lint → Test → Build Docker
│   │   └── db-migration.yml
│   ├── src/
│   │   ├── config/env.js
│   │   ├── models/Patient.js
│   │   ├── controllers/patientController.js
│   │   ├── services/patientService.js
│   │   ├── routes/patientRoutes.js
│   │   ├── middleware/
│   │   └── index.js
│   ├── migrations/
│   ├── seeders/
│   ├── deployment/ecs/
│   ├── docker-compose.yml
│   ├── package.json
│   ├── .env
│   ├── .env.example
│   └── README.md
│
├── AppointmentService/                # Microservice #2
│   ├── .github/workflows/
│   │   └── ci-build.yml
│   ├── src/
│   │   ├── config/
│   │   ├── models/AppointmentSlot.js
│   │   ├── controllers/
│   │   ├── services/
│   │   ├── routes/
│   │   └── index.js
│   ├── deployment/ecs/
│   ├── package.json
│   ├── .env
│   └── README.md
│
├── patient-portal/                     # Patient Portal (Web UI)
│   ├── .github/workflows/
│   │   └── ci-build.yml
│   ├── src/
│   │   ├── services/api.js
│   │   ├── hooks/useAPI.js
│   │   ├── pages/Dashboard.jsx
│   │   ├── components/
│   │   ├── main.jsx
│   │   └── App.jsx
│   ├── public/
│   ├── nginx.conf
│   ├── vite.config.js
│   ├── package.json
│   ├── .env
│   └── README.md
│
└── Infrastructure/                    # IaC - AWS ECS Fargate
    ├── .github/workflows/
    │   └── cd-deployment.yml          # Terraform Plan → Apply
    ├── terraform/
    │   ├── modules/
    │   │   ├── ecs/main.tf
    │   │   ├── rds/main.tf
    │   │   ├── ecr/main.tf
    │   │   ├── networking/main.tf
    │   │   └── secrets/main.tf
    │   └── environments/
    │       ├── dev/
    │       │   ├── provider.tf
    │       │   ├── variables.tf
    │       │   └── main.tf
    │       └── prod/
    │           ├── provider.tf
    │           ├── variables.tf
    │           └── main.tf
    ├── .gitignore
    └── README.md
```

---

## 🔄 CI/CD Flow

### Service CI Pipelines (Automatic)

```
Push Code → Lint → Test → Security Scan → Build Docker → Push to GHCR
```

**Services:** Patient Service, Appointment Service, Frontend
**Trigger:** Any push to any branch
**Duration:** 10-15 minutes
**Output:** Docker images tagged with branch/commit SHA

### Infrastructure CD Pipeline (Semi-Manual)

```
Push to main → Terraform Init → Validate → Plan → [Manual Approval] → Apply
```

**Trigger:** Push to Infrastructure repo main branch (or manual trigger)
**Environments:** Dev (auto-apply), Prod (requires approval)
**Duration:** 5-10 minutes plan, 5 minutes apply
**Output:** AWS resources deployed via CloudFormation

---

## 🗄️ Database Configuration

### MySQL Setup

All services configured for **MySQL 8.0** on port 3306

```sql
-- Databases
CREATE DATABASE hospital_patient_db CHARACTER SET utf8mb4;
CREATE DATABASE hospital_appointment_db CHARACTER SET utf8mb4;

-- Run migrations
cd PatientService
npm run db:migrate             # Patient schema

cd AppointmentService
npm run db:migrate             # Appointment schema

-- Verify
SHOW TABLES IN hospital_patient_db;
SELECT * FROM patients LIMIT 1;
```

### Local MySQL (Required for local development)

```bash
# Docker approach (easiest)
docker run --name mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 3306:3306 \
  -d mysql:8.0

# Test connection
mysql -h localhost -u root -p
```

---

## 📚 Documentation Guide

This repository now keeps Markdown documentation focused on `README.md` files:
- Root `README.md` for overall architecture, setup, and deployment flow
- `PatientService/README.md` for patient service usage
- `AppointmentService/README.md` for appointment service usage
- `patient-portal/README.md` for frontend usage
- `Infrastructure/README.md` for Terraform and delivery pipeline

---

## 🔐 Security Features

✅ **Secrets Management**
- AWS Secrets Manager for credentials
- GitHub secrets for CI/CD
- No hardcoded secrets in code

✅ **Access Control**
- IAM roles with least privilege
- OIDC authentication for GitHub Actions
- RDS encryption enabled
- VPC security groups

✅ **Code Quality**
- ESLint + Prettier for code formatting
- npm audit for vulnerability scanning
- Snyk integration available
- GitHub branch protection rules

✅ **Audit & Logging**
- CloudWatch logs for all services
- CloudTrail for AWS API calls
- VPC Flow Logs enabled
- RDS audit logs available

---

## 📊 AWS Resources Created

### Compute
- ECS Fargate cluster with 3 services
- Auto Scaling groups (min: 2, max: 10)
- Application Load Balancer with target groups

### Database
- RDS MySQL 8.0 instance (db.t3.small/medium)
- Multi-AZ failover in production
- Automated backups (30-day retention)
- Encrypted storage

### Networking
- VPC with CIDR 10.0.0.0/16
- 2 public subnets (for ALB)
- 2 private subnets (for services & RDS)
- Internet Gateway, NAT Gateway
- Security groups with inbound/outbound rules

### Registry & Secrets
- 3 ECR repositories (patient, appointment, frontend)
- AWS Secrets Manager for database credentials
- Parameter Store for configuration

### Monitoring
- CloudWatch Logs for services
- CloudWatch Metrics for performance
- CloudWatch Alarms (configured)
- Cost allocation tags

---

## 💰 Cost Estimation

| Resource | Estimated Monthly Cost |
|----------|------------------------|
| ECS Fargate (3 services, 2 tasks each) | $40 |
| RDS MySQL (db.t3.small) | $30 |
| Application Load Balancer | $15 |
| Data Transfer & Storage | $10 |
| CloudWatch Logs | $5 |
| **Total** | **~$100** |

*Costs vary by region and usage. Using AWS Pricing Calculator for accurate estimates.*

---

## 🚀 Deployment Steps

### Phase 1: Preparation (Day 1)
- [ ] Configure GitHub repositories, secrets, branch protections, and OIDC role
- [ ] Create 4 GitHub repositories
- [ ] Push code to all repositories

### Phase 2: AWS Setup (Day 2)
- [ ] Create S3 + DynamoDB for Terraform state
- [ ] Create GitHub Actions IAM role
- [ ] Verify AWS credentials and permissions

### Phase 3: Infrastructure (Day 2-3)
- [ ] Push Infrastructure repo
- [ ] Review Terraform plan
- [ ] Approve and apply infrastructure
- [ ] Monitor resource creation

### Phase 4: Verification (Day 3-4)
- [ ] Test all API endpoints
- [ ] Verify database connectivity
- [ ] Load testing
- [ ] Security scanning

### Phase 5: Production (Day 4-5)
- [ ] Configure domain/DNS
- [ ] Enable SSL certificate
- [ ] Set up monitoring and alerts
- [ ] Team training and handoff

---

## 🔧 Common Operations

### Deploy New Version

```bash
# Make code changes
git add .
git commit -m "feat: new feature"
git push origin main

# CI pipeline automatically:
# 1. Runs tests
# 2. Builds Docker image
# 3. Pushes to GHCR
# 4. ECS auto-updates service
```

### Scale Service

```bash
# Manually scale
aws ecs update-service \
  --cluster hospital-prod-cluster \
  --service patient-service \
  --desired-count 5

# Auto-scaling handles spikes automatically
```

### View Logs

```bash
# Follow logs in real-time
aws logs tail /ecs/patient-service --follow

# Grep for errors
aws logs tail /ecs/patient-service --follow | grep -i error
```

### Rollback Deployment

```bash
# Get previous task definition
aws ecs describe-task-definition --task-definition patient-service

# Update service with previous version
aws ecs update-service \
  --cluster hospital-prod-cluster \
  --service patient-service \
  --task-definition patient-service:PREVIOUS_VERSION \
  --force-new-deployment
```

---

## ✅ Pre-Deployment Checklist

- [ ] All 4 repositories created and code pushed
- [ ] GitHub secrets configured for all repos
- [ ] AWS resources created (S3, DynamoDB, IAM)
- [ ] CI pipelines tested successfully
- [ ] Local development tested
- [ ] Database migrations run locally
- [ ] Docker images build successfully
- [ ] Terraform plan reviewed and approved
- [ ] Team trained on operations
- [ ] Monitoring and alerting configured
- [ ] Backup and recovery procedure tested

---

## 🆘 Troubleshooting

### Issue: CI Pipeline Fails

**Solution:** Check GitHub Actions logs
```bash
# Repository → Actions → Failed workflow → View logs
# Common causes: npm install fails, linting errors, Docker build fails
```

### Issue: Terraform Apply Fails

**Solution:** Validate configuration
```bash
cd terraform/environments/prod
terraform validate
terraform plan -out=tfplan
```

### Issue: ECS Task Won't Start

**Solution:** Check CloudWatch logs
```bash
aws logs tail /ecs/patient-service --follow
# Look for: database connection errors, missing environment variables
```

### Issue: Database Connection Error

**Solution:** Verify RDS accessibility
```bash
# Test from bastion/EC2
mysql -h RDS_ENDPOINT -u admin -p hospital_patient_db

# Check security group
aws ec2 describe-security-groups --group-names hospital-rds-sg
```

If issues continue, check CI logs, ECS task logs, and Terraform output for the failing step.

---

## 📞 Support & Resources

- **AWS Documentation:** https://docs.aws.amazon.com/
- **Terraform Docs:** https://www.terraform.io/
- **GitHub Actions:** https://docs.github.com/en/actions
- **Node.js + Express:** https://expressjs.com/
- **React Documentation:** https://react.dev/
- **MySQL Documentation:** https://dev.mysql.com/doc/

---

## 📝 Project Status

| Component | Status | Tests | Deployment |
|-----------|--------|-------|------------|
| Patient Service | ✅ Complete | ✅ Passing | 🟡 Ready |
| Appointment Service | ✅ Complete | ✅ Passing | 🟡 Ready |
| Frontend | ✅ Complete | ✅ Passing | 🟡 Ready |
| Infrastructure | ✅ Complete | ✅ Validated | 🟡 Ready |
| CI/CD Pipelines | ✅ Complete | ✅ Tested | ✅ Active |

**Legend:** ✅ Complete | 🟡 Ready for Deployment | 🟢 Deployed

---

## 🎯 Next Steps

1. **Set up GitHub** - Create 4 repositories, configure secrets, and enable branch protections
2. **Run deployment checklist** - Execute phases in this README and validate each checkpoint
3. **Monitor CI/CD pipelines** - Verify all workflows execute successfully
4. **Test API endpoints** - Validate all services are functional
5. **Configure monitoring** - Set up CloudWatch dashboards and alarms
6. **Production handoff** - Team training and documentation

---

## 📄 Files Included

```
📁 Patient/
├── 📄 README.md (this file)
│
├── 📁 PatientService/
│   ├── Source code, models, migrations
│   ├── CI/CD workflows
│   └── README.md
│
├── 📁 AppointmentService/
│   ├── Appointment management service
│   ├── CI/CD pipeline
│   └── README.md
│
├── 📁 patient-portal/
│   ├── React application
│   ├── Responsive UI
│   └── README.md
│
└── 📁 Infrastructure/
    ├── Terraform modules and environments
    ├── CD pipeline
    └── README.md
```

**CI/CD Pipelines:** 4
**AWS Services:** 9+

---

## 📞 Questions or Issues?

Refer to the appropriate documentation:
- **Local Development:** See `PatientService/README.md`
- **Infrastructure:** See `Infrastructure/README.md`

---

**Status:** ✅ Production-Ready
**Last Updated:** 2024
**Version:** 1.0.0

🚀 **Ready to Deploy!**
