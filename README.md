# 3-Tier DevSecOps Project + Terraform Infrastructure

![DevSecOps Pipeline](https://img.shields.io/badge/DevSecOps-Pipeline-blue) ![AWS](https://img.shields.io/badge/AWS-EKS-orange) ![Kubernetes](https://img.shields.io/badge/Kubernetes-1.34-blue) ![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-red) ![Terraform](https://img.shields.io/badge/Terraform-IaC-purple)

## 🚀 Project Overview

A comprehensive **DevSecOps mega project** combining a **3-tier web application** with **Infrastructure as Code (Terraform)** deployed on **AWS EKS**. This project demonstrates enterprise-level cloud-native development with automated CI/CD pipelines, security scanning, and production-ready infrastructure provisioning.

## 🏗️ Complete Architecture

### **Application Architecture**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │    Backend      │    │    Database     │
│   (React/Node)  │◄──►│   (Node.js)     │◄──►│    (MySQL)      │
│   Port: 80      │    │   Port: 5000    │    │   Port: 3306    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### **Infrastructure Architecture**
```
┌─────────────────────────────────────────────────────────────────┐
│                           AWS Cloud                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐ │
│  │   VPC Network   │  │   EKS Cluster   │  │   RDS/Storage   │ │
│  │   - Subnets     │  │   - Worker Nodes│  │   - EBS Volumes │ │
│  │   - Security    │  │   - Load Balancer│  │   - Backups     │ │
│  │   - NAT Gateway │  │   - Auto Scaling│  │   - Encryption  │ │
│  └─────────────────┘  └─────────────────┘  └─────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ Terraform Provisioning
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CI/CD Pipeline                              │
│  GitHub → Jenkins → SonarQube → Docker → Kubernetes → Monitor  │
└─────────────────────────────────────────────────────────────────┘
```

## 📦 Projects Included

### 🎯 **3-Tier DevSecOps Application**
Complete full-stack web application with:
- **Frontend**: React/Node.js user interface
- **Backend**: RESTful API with Node.js/Express
- **Database**: MySQL with persistent storage
- **CI/CD**: Jenkins pipeline with automated testing
- **Security**: SonarQube integration and vulnerability scanning

### 🏗️ **Terraform Infrastructure Project**
Infrastructure as Code implementation featuring:
- **Modular Design**: Reusable Terraform modules
- **Multi-Environment**: Dev, staging, and production configs
- **AWS Resources**: VPC, EKS, EC2, RDS, S3, IAM
- **State Management**: Remote state with S3 backend
- **Security**: Best practices with least privilege access

## 🛠️ Technologies Used

### **Infrastructure & Cloud**
- **AWS EKS** - Kubernetes cluster management
- **AWS EC2** - Jenkins server hosting
- **AWS ALB** - Application Load Balancer
- **AWS EBS** - Persistent storage
- **Terraform** - Infrastructure as Code provisioning
- **AWS VPC** - Network isolation and security

### **DevOps & CI/CD**
- **Jenkins** - Automated CI/CD pipelines
- **Docker** - Containerization
- **Kubernetes** - Container orchestration
- **GitHub Webhooks** - Automated triggers

### **Security & Quality**
- **SonarQube** - Static code analysis
- **Cert-Manager** - SSL certificate management
- **Let's Encrypt** - Free SSL certificates
- **NGINX Ingress** - Secure routing

### **Application Stack**
- **Frontend**: Node.js/React
- **Backend**: Node.js REST API
- **Database**: MySQL with persistent volumes

## 📋 Prerequisites

- AWS Account with appropriate permissions
- GitHub account
- Basic knowledge of Kubernetes and Docker
- Jenkins server setup

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/3-tier-devsecops-project.git
cd 3-tier-devsecops-project
```

### 2. Infrastructure Setup (Terraform)
```bash
cd Mega-Project-Terraform

# Initialize Terraform
terraform init

# Plan infrastructure
terraform plan

# Apply infrastructure
terraform apply
```

### 3. Application Deployment
```bash
cd 3-Tier-DevSecOps-Mega-Project

# Deploy to Kubernetes
kubectl apply -f k8s/

# Or use the enhanced production manifests
kubectl apply -f ../k8s-prod/
```

### 4. Configure Jenkins Pipeline
- Import the `Jenkinsfile` into your Jenkins instance
- Configure GitHub webhook with your Jenkins URL
- Set up required credentials (Docker Hub, AWS, SonarQube)

### 5. Local Development
```bash
# Run with Docker Compose
docker-compose up -d

# Access application at http://localhost:3000
```

## 📁 Project Structure

```
3-tier-devsecops-project/
├── 3-Tier-DevSecOps-Mega-Project/    # Complete 3-tier application source code
│   ├── frontend/                      # React/Node.js frontend application
│   ├── backend/                       # Node.js REST API backend
│   ├── k8s/                          # Kubernetes manifests
│   ├── Jenkinsfile                   # Original Jenkins pipeline
│   └── README.md                     # 3-tier project documentation
├── Mega-Project-Terraform/           # Infrastructure as Code with Terraform
│   ├── modules/                      # Terraform reusable modules
│   ├── environments/                 # Environment-specific configurations
│   ├── main.tf                       # Main Terraform configuration
│   ├── variables.tf                  # Variable definitions
│   └── README.md                     # Terraform project documentation
├── k8s-prod/                         # Production Kubernetes manifests
│   ├── mysql.yaml                    # MySQL database deployment
│   ├── backend.yaml                  # Backend service deployment
│   ├── frontend.yaml                 # Frontend service deployment
│   ├── ingress.yaml                  # NGINX ingress configuration
│   └── sc.yaml                       # Storage class for EBS
├── Jenkinsfile                       # Enhanced CI/CD pipeline configuration
├── Dockerfile                        # Multi-stage Docker build
├── docker-compose.yml                # Local development setup
├── sonar-project.properties          # SonarQube configuration
├── LICENSE                           # MIT License
└── README.md                         # Main project documentation
```

## 🔄 CI/CD Pipeline

The Jenkins pipeline includes the following stages:

1. **📥 Checkout** - Pull latest code from GitHub
2. **🔍 Code Analysis** - SonarQube static analysis
3. **🏗️ Build** - Docker image creation
4. **📤 Push** - Push to Docker Hub registry
5. **✅ Approval** - Manual production deployment approval
6. **🚀 Deploy** - Kubernetes deployment to EKS
7. **🔍 Verify** - Health checks and validation

## 🌐 Access the Application

After successful deployment:

- **Load Balancer URL**: `http://your-alb-url.elb.amazonaws.com`
- **Custom Domain**: `https://yourdomain.com` (if configured)

## 🔧 Configuration

### Environment Variables
```bash
# Backend Configuration
DB_HOST=mysql
DB_USER=root
DB_PASSWORD=your-password
DB_NAME=devopsshack

# Frontend Configuration
REACT_APP_API_URL=http://backend-svc:5000
```

### Jenkins Credentials Required
- `docker-cred` - Docker Hub username/password
- `k8-prod-token` - Kubernetes cluster access token
- `sonar-token` - SonarQube authentication token

## 🛡️ Security Features

- **SSL/TLS Encryption** with Let's Encrypt certificates
- **Static Code Analysis** with SonarQube integration
- **Container Security** with minimal base images
- **Network Policies** for pod-to-pod communication
- **RBAC** for Kubernetes access control

## 📊 Monitoring & Logging

- **Kubernetes Dashboard** for cluster monitoring
- **Jenkins Build Logs** for pipeline tracking
- **Application Logs** via kubectl logs
- **Ingress Metrics** through NGINX controller

## 🚨 Troubleshooting

### Common Issues

1. **Pipeline Fails at Kubernetes Deployment**
   ```bash
   # Check cluster connectivity
   kubectl cluster-info
   
   # Verify node status
   kubectl get nodes
   ```

2. **Application Not Accessible**
   ```bash
   # Check ingress status
   kubectl get ingress -n prod
   
   # Verify service endpoints
   kubectl get svc -n prod
   ```

3. **SSL Certificate Issues**
   ```bash
   # Check cert-manager status
   kubectl get certificates -n prod
   
   # View certificate details
   kubectl describe certificate -n prod
   ```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Original Project Inspiration**: [Aditya Jaiswal](https://github.com/adityajaiswal) - Special thanks for the foundational architecture and guidance
- **AWS Documentation** - For comprehensive EKS guides
- **Kubernetes Community** - For excellent documentation and support
- **Jenkins Community** - For robust CI/CD capabilities

## 📞 Contact

**Project Maintainer**: [Your Name]
- GitHub: [@your-username](https://github.com/your-username)
- LinkedIn: [Your LinkedIn Profile](https://linkedin.com/in/your-profile)
- Email: your.email@example.com

---

⭐ **If this project helped you, please give it a star!** ⭐

---

## 📈 Project Stats

![GitHub stars](https://img.shields.io/github/stars/your-username/3-tier-devsecops-project)
![GitHub forks](https://img.shields.io/github/forks/your-username/3-tier-devsecops-project)
![GitHub issues](https://img.shields.io/github/issues/your-username/3-tier-devsecops-project)
![GitHub license](https://img.shields.io/github/license/your-username/3-tier-devsecops-project)
