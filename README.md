# 3-Tier Cloud-Native DevSecOps Platform

![DevSecOps](https://img.shields.io/badge/DevSecOps-Pipeline-blue)
![AWS EKS](https://img.shields.io/badge/AWS-EKS-orange)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.28-blue)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red)
![Prometheus](https://img.shields.io/badge/Prometheus-Monitoring-orange)
![Grafana](https://img.shields.io/badge/Grafana-Dashboards-yellow)

---

## Overview

A production-grade, end-to-end cloud-native platform built entirely from scratch.

This is not a tutorial follow-along. Every architectural decision — from VPC design to SLO definitions — was made deliberately and independently. The goal was to build the kind of infrastructure that would survive a banking-sector security audit: IAM least-privilege throughout, secrets never in plaintext, every alert actionable, every deployment rollback-capable.

**What this demonstrates end-to-end:**
- Zero-ClickOps AWS infrastructure via Terraform modules
- Commit-to-production CI/CD with automated rollback
- Production Kubernetes on EKS with HPA, RBAC, and network policies
- Full observability: Prometheus + Grafana (SLI/SLO tracked) + ELK log aggregation
- DevSecOps: IAM scoping, Secrets Manager, Trivy scanning, pipeline security gates
- Python automation: deployment validators, drift detectors, health checks

---

## Architecture

### Full System Architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│                         AWS Cloud                                    │
│                                                                      │
│  ┌──────────┐     ┌───────────────┐     ┌──────────────────────┐    │
│  │ Route 53 │────▶│  CloudFront   │────▶│      ALB / ELB       │    │
│  └──────────┘     │  (CDN Layer)  │     └──────────┬───────────┘    │
│                   └───────────────┘                │                │
│                                                    ▼                │
│          ┌─────────────────────────────────────────────────────┐    │
│          │                  AWS EKS Cluster                     │    │
│          │                                                      │    │
│          │  ┌─────────────┐  ┌──────────────┐  ┌───────────┐  │    │
│          │  │  Tier 1     │  │  Tier 2      │  │  Tier 3   │  │    │
│          │  │  React.js   │  │  FastAPI     │  │  RDS      │  │    │
│          │  │  Frontend   │◄─┤  Backend     │◄─┤  (MySQL/  │  │    │
│          │  │  (Port: 80) │  │  (Port:8000) │  │  Postgres)│  │    │
│          │  └─────────────┘  └──────────────┘  └───────────┘  │    │
│          │                                                      │    │
│          │  HPA Autoscaling · Rolling Updates · RBAC · Probes  │    │
│          │  Network Policies · Pod Disruption Budgets           │    │
│          └──────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Observability Stack                                         │    │
│  │  Prometheus  ·  Grafana  ·  Alertmanager  ·  ELK/OpenSearch │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                      │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │  Infrastructure as Code — Terraform                          │    │
│  │  VPC · EC2 · EKS · RDS · S3 · IAM · ALB · CloudFront        │    │
│  │  Lambda · SNS/SQS · Secrets Manager · Route 53               │    │
│  └─────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────────────────────┘
```

### CI/CD Pipeline Flow

```
Code Push / PR
      │
      ▼
┌─────────────────────┐
│   GitHub Actions    │  Trigger, PR validation
└──────────┬──────────┘
           │
      ┌────▼────┐
      │ Jenkins │
      └────┬────┘
           │
    ┌──────▼──────────────┐
    │ Stage 1: Checkout   │  Git clone, branch validation
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 2: Code Scan  │  SonarQube static analysis
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 3: Build      │  Docker multi-stage build
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 4: Sec Scan   │  Trivy image scan — critical CVEs block pipeline
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 5: Push       │  ECR push with SHA-based immutable tags
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 6: Deploy     │  Helm upgrade to EKS (rolling, zero-downtime)
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 7: Verify     │  Python deployment validator — health endpoints
    └──────┬──────────────┘
           │
    ┌──────▼──────────────┐
    │ Stage 8: Rollback   │  Auto rollback on verification failure
    └─────────────────────┘
```

---

## Tech Stack

| Layer | Tools |
|-------|-------|
| **Cloud** | AWS — EC2, EKS, ECS, ECR, RDS, S3, VPC, IAM, ALB/ELB, CloudFront, Lambda, SNS, SQS, Secrets Manager, Route 53, OpenSearch, CloudWatch, CloudTrail |
| **IaC** | Terraform (modules, remote state, workspaces, multi-env), Ansible, CloudFormation |
| **CI/CD** | Jenkins, GitHub Actions, ArgoCD |
| **Containers** | Docker (multi-stage builds), Kubernetes/EKS, Helm |
| **Observability** | Prometheus, Grafana, Alertmanager, ELK/OpenSearch, CloudWatch |
| **Security** | IAM least-privilege, K8s RBAC, AWS Secrets Manager, Trivy, SonarQube, pipeline security gates |
| **Application** | Python FastAPI (backend), React.js (frontend) |
| **Scripting** | Python, Bash |
| **Database** | MySQL (RDS), Redis (ElastiCache) |

---

## Repository Structure

```
3-tier-devsecops-project/
│
├── terraform/                         # All AWS infrastructure as code
│   ├── modules/
│   │   ├── vpc/                       # VPC, subnets, IGW, NAT, route tables
│   │   ├── eks/                       # EKS cluster, node groups, OIDC
│   │   ├── rds/                       # RDS instance, parameter groups
│   │   ├── s3/                        # Buckets, policies, lifecycle rules
│   │   ├── iam/                       # Roles, policies, instance profiles
│   │   ├── alb/                       # Load balancer, target groups, listeners
│   │   └── cloudfront/                # CDN distribution, cache behaviours
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── backend.tf                     # Remote state — S3 bucket + DynamoDB lock
│
├── kubernetes/
│   ├── helm/
│   │   ├── frontend/                  # React frontend Helm chart
│   │   ├── backend/                   # FastAPI backend Helm chart
│   │   └── values/                    # values-dev.yaml, values-prod.yaml
│   └── manifests/
│       ├── namespaces/
│       ├── rbac/                      # ServiceAccounts, Roles, RoleBindings
│       ├── network-policies/          # Service-to-service communication rules
│       └── hpa/                       # HorizontalPodAutoscaler configs
│
├── ci-cd/
│   ├── Jenkinsfile                    # Multi-stage Jenkins pipeline
│   └── .github/workflows/
│       ├── ci.yml                     # Build, test, scan, push to ECR
│       └── deploy.yml                 # Deploy to EKS via Helm
│
├── observability/
│   ├── prometheus/                    # Scrape configs, alerting rules
│   ├── grafana/                       # Dashboard JSON exports (SLI/SLO)
│   ├── alertmanager/                  # Routing config, receivers
│   └── elk/                           # Logstash pipelines, index templates
│
├── application/
│   ├── frontend/                      # React.js application + Dockerfile
│   └── backend/                       # Python FastAPI application + Dockerfile
│
├── scripts/
│   ├── deploy-validator.py            # Post-deploy health check automation
│   ├── drift-detector.py              # Terraform state vs live AWS drift check
│   └── env-health-check.sh           # Pre-deploy environment readiness check
│
└── docs/
    ├── architecture.md
    ├── runbooks/
    ├── sops/
    └── troubleshooting/
```

---

## Infrastructure — Terraform

Zero ClickOps. Every AWS resource provisioned through Terraform modules.

**Networking**
- Custom VPC with public/private subnets across 3 Availability Zones
- Internet Gateway, NAT Gateway per AZ, route tables
- Security groups with least-privilege ingress/egress
- NACLs for subnet-level traffic control

**Compute & Orchestration**
- EKS cluster with managed node groups (on-demand + spot mix)
- Auto Scaling Groups with launch templates
- EC2 instances for Jenkins — provisioned via Terraform, not manually clicked

**Data & Storage**
- RDS (MySQL) — Multi-AZ, automated backups, encryption at rest
- S3 buckets — versioning, lifecycle policies, server-side encryption
- ElastiCache (Redis) — session management and caching layer

**Security**
- IAM roles scoped per service — no wildcard policies
- AWS Secrets Manager for all credentials — no plaintext secrets anywhere
- CloudTrail enabled — full API audit logging
- Remote Terraform state in S3 with DynamoDB state locking

**Delivery**
- CloudFront distribution for frontend CDN
- ALB with HTTPS termination and path-based routing
- Route 53 hosted zone and DNS records

---

## Kubernetes Operations

**Cluster Design**
- Separate namespaces per environment (dev / staging / prod)
- RBAC with service account scoping — no wildcard permissions
- Network policies enforcing service-to-service communication rules
- Resource requests and limits on all workloads

**Deployment Strategy**
- Rolling updates — `maxSurge: 1`, `maxUnavailable: 0` — zero-downtime deploys
- Liveness and readiness probes on every pod
- HPA (Horizontal Pod Autoscaler) on frontend and backend
- Pod Disruption Budgets protecting availability during node operations

**Helm**
- Separate charts for frontend, backend, and observability stack
- Environment-specific value files (`values-dev.yaml`, `values-prod.yaml`)
- Versioned releases — every deploy is rollback-capable in one command

---

## Observability

### Metrics — Prometheus + Grafana
- Prometheus scraping all services, Kubernetes components, and node exporters
- Custom alerting rules: p99 latency, error rate, pod crash loops, disk pressure, memory pressure
- Alertmanager routing: critical → PagerDuty, warning → Slack, info → email
- Grafana dashboards: service health, SLI/SLO tracking, error budget burn rate, infrastructure overview

### Logs — ELK / OpenSearch
- Fluent Bit DaemonSet shipping container logs to OpenSearch
- Structured JSON logging from application layer
- Index lifecycle management — hot/warm/cold tiering
- Kibana dashboards for log-based alerting and incident triage

### SLI / SLO Definitions

| Service | SLI | SLO Target |
|---------|-----|-----------|
| Frontend | Availability (HTTP 2xx rate) | 99.9% |
| Backend API | Latency (p99 < 300ms) | 99.5% |
| Backend API | Error rate (5xx < 0.1%) | 99.9% |
| Pipeline | Successful deploy rate | 99% |

---

## Security — DevSecOps at Every Layer

| Layer | Control |
|-------|---------|
| **IAM** | Least-privilege per pipeline stage — no wildcard policies, no shared credentials |
| **Secrets** | AWS Secrets Manager — nothing in code, ENV vars, or container logs |
| **Images** | Trivy scan in CI — critical CVEs block the pipeline before push |
| **Code** | SonarQube static analysis — quality gate blocks on critical findings |
| **Kubernetes** | RBAC, network policies, pod security standards |
| **Pipeline** | Credential isolation per environment — no cross-environment access |
| **Audit** | CloudTrail on all AWS API calls, Kubernetes audit logging enabled |
| **TLS** | Cert-Manager with Let's Encrypt — all ingress traffic encrypted |

---

## Python Automation Scripts

| Script | Purpose |
|--------|---------|
| `deploy-validator.py` | Polls health endpoints post-deploy — fails pipeline on degraded state, triggers rollback |
| `drift-detector.py` | Compares Terraform state against live AWS — alerts on configuration drift |
| `env-health-check.sh` | Full environment readiness verification before deployment proceeds |

---

## How to Deploy

### Prerequisites
- AWS CLI configured with appropriate IAM permissions
- Terraform >= 1.5
- kubectl >= 1.28
- Helm >= 3.12
- Docker

### 1. Provision Infrastructure
```bash
cd terraform/environments/dev
terraform init
terraform plan -var-file=terraform.tfvars
terraform apply -var-file=terraform.tfvars
```

### 2. Configure kubectl
```bash
aws eks update-kubeconfig --region eu-west-2 --name devops-platform-dev
kubectl get nodes
```

### 3. Deploy Observability Stack
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm upgrade --install monitoring prometheus-community/kube-prometheus-stack \
  -f observability/prometheus/values.yaml \
  -n monitoring --create-namespace
```

### 4. Deploy Application
```bash
# Backend
helm upgrade --install backend ./kubernetes/helm/backend \
  -f kubernetes/helm/backend/values-dev.yaml \
  -n application --create-namespace

# Frontend
helm upgrade --install frontend ./kubernetes/helm/frontend \
  -f kubernetes/helm/frontend/values-dev.yaml \
  -n application
```

### 5. Trigger CI/CD
Push to `main` — Jenkins webhook fires and handles everything from build to verified deployment.

---

## Troubleshooting

**Pipeline fails at Kubernetes deploy**
```bash
kubectl cluster-info
kubectl get nodes
kubectl describe pod <pod-name> -n application
```

**Application not accessible**
```bash
kubectl get ingress -n application
kubectl get svc -n application
kubectl logs -l app=backend -n application --tail=50
```

**SSL certificate issues**
```bash
kubectl get certificates -n application
kubectl describe certificate <cert-name> -n application
```

**Terraform drift detected**
```bash
cd terraform/environments/prod
terraform plan
python scripts/drift-detector.py
```

---

## Documentation

Full documentation in `/docs`:
- `architecture.md` — Design decisions and rationale
- `runbooks/` — Incident response step-by-step guides
- `sops/` — Standard operating procedures
- `troubleshooting/` — Common failure patterns and resolutions

---

## Author

**Abhishek Singh** — DevOps Engineer · SRE · Platform Engineer

3 years of production infrastructure experience at Lloyds Banking Group — one of the UK's most regulated banking environments.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://linkedin.com/in/abhishek-singh-2b96961a0)
[![Portfolio](https://img.shields.io/badge/Portfolio-000000?style=flat&logo=vercel&logoColor=white)](https://portfolio-abhi002sheks-projects.vercel.app)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white)](mailto:itsabhishek1531@gmail.com)

---

*📍 Hyderabad, India · Available immediately · Open to DevOps / SRE / Cloud / Platform Engineer roles*
