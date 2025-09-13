# **Cloud-Native DevOps Project** 🚀

![Project Banner](https://imgur.com/cp6cwlX.png)

This repository contains a full-stack application deployment using modern DevOps practices and cloud-native technologies. The project demonstrates the implementation of Infrastructure as Code (IaC), containerization, orchestration, and continuous integration/deployment (CI/CD) pipelines.

## 📋 **Project Overview**

This project implements a complete DevOps lifecycle for a cloud-native application with:
- Infrastructure automation using Terraform
- Container orchestration with Kubernetes (EKS)
- CI/CD implementation using Jenkins
- Artifact management with Nexus
- Code quality with SonarQube
- Security scanning with CodeQL and Veracode
- Monitoring and observability

## 🏗️ **Infrastructure as Code - Terraform**

The infrastructure is completely automated using Terraform with state management and locking enabled through AWS S3.

### **Resources Created:**
- **VPC Architecture**
  - Public Subnet: Hosts Bastion Host, VPN, and ALB (Ingress Controller)
  - Private Subnet: Houses EKS Cluster
  - DB Subnet: Contains RDS (MySQL)
  - CIDR blocks properly segmented for each subnet
  - NAT Gateway for private subnet internet access
  - Internet Gateway for public subnet
- **Additional AWS Services**
  - Route53 for DNS management and service discovery
  - CloudFront CDN for static content delivery
  - EFS for persistent storage with proper mount targets
  - Amazon ECR for secure container registry
  - S3 buckets for artifact storage and Terraform state
  - KMS for encryption key management

### **Terraform Structure**
```
terraform/
├── 00-vpc/          # VPC and networking
├── 10-sg/           # Security Groups
├── 20-bastion/      # Bastion Host
├── 30-db/           # RDS Database
├── 40-eks/          # EKS Cluster
├── 50-acm/          # SSL Certificates
├── 60-ingress-alb/  # ALB Ingress
└── 70-ecr/          # Container Registry
```

> [!NOTE]
> For detailed infrastructure setup instructions, please refer to the [Infrastructure Setup Guide](terraform/README.md).

## ☸️ **Kubernetes Architecture - EKS**

Our application runs on Amazon EKS (Elastic Kubernetes Service) with the following setup:

### **Cluster Configuration**
- EKS version: 1.24+
- Node Groups: Mix of on-demand and spot instances
- Auto-scaling enabled (2-10 nodes)
- Multi-AZ deployment for high availability

### **Components**
- **Traffic Flow**
  - AWS Application Load Balancer (ALB) as entry point
  - Ingress Controller for traffic routing
    - URL path-based routing
    - SSL termination
    - Rate limiting
  - Kubernetes Services
    - ClusterIP for internal communication
    - NodePort for debugging
    - LoadBalancer for external services
- **Application Management**
  - Deployments
    - Rolling updates strategy
    - Resource limits and requests
    - Health checks and readiness probes
  - ConfigMaps
    - Environment-specific configurations
    - Feature flags
    - Application settings
  - Secrets
    - Credentials management
    - Sensitive configuration
  - Helm Charts
    - Application packaging
    - Version management
    - Dependency handling
  - Storage
    - EFS StorageClass
    - PersistentVolumeClaims
    - Dynamic provisioning

### **Helm Chart Structure**
```
helm/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── configmap.yaml
    ├── secret.yaml
    └── hpa.yaml
```

## 🚀 **CI/CD Pipeline - Jenkins**

The continuous integration and deployment pipeline is implemented using Jenkins, triggered by GitHub webhooks.

### **Pipeline Architecture**
- Multi-branch pipeline
- Shared libraries for common functions
- Parallel execution where possible
- Timeout and retry mechanisms
- Slack/Email notifications

### **Pipeline Stages:**
1. **Build Initialization**
   - Dependency installation
   - Code checkout
   - Environment validation
   - Cache restoration
2. **Code Quality**
   - SonarQube analysis
     - Code coverage requirements
     - Security hotspots
     - Code smells
   - Code coverage reports
   - Unit tests
   - Integration tests
3. **Infrastructure**
   - Terraform plan and apply
   - Infrastructure validation
   - Security group verification
   - Network connectivity tests
4. **Containerization**
   - Multi-stage Dockerfile builds
   - Docker image build
     - Layer optimization
     - Security scanning
   - Push to Amazon ECR
   - Image scanning
5. **Deployment**
   - Helm chart validation
   - Kubernetes manifest generation
   - Rolling deployment
   - Smoke tests
   - Rollback procedures

### **Jenkinsfile Structure**
```groovy
pipeline {
    agent {
        label 'AGENT-1'
    }
    environment {
        // Environment variables
    }
    stages {
        stage('Build') {
            // Build stage
        }
        stage('Test') {
            // Test stage
        }
        // Additional stages
    }
    post {
        // Post-build actions
    }
}
```

## 🛠️ **Setup Instructions**

### **Prerequisites**
- AWS Account with appropriate permissions
- Domain name for application
- GitHub repository
- Docker installed locally
- kubectl and helm installed
- Terraform installed

### 1. **Jenkins Setup**
1. Create EC2 instance for Jenkins
   - Instance type: t3.large (minimum)
   - Storage: 30GB+ EBS
   - Security Group: Ports 22, 8080
2. Execute the setup script:
   ```bash
   sh jenkins.sh
   ```
3. Access Jenkins UI at `http://<jenkins-ip>:8080`
4. Follow initial setup wizard using the password from:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
5. Install required plugins:
   - Pipeline
   - Git
   - Docker
   - Kubernetes
   - SonarQube Scanner
   - Nexus Artifact Uploader

### 2. **Jenkins Agent Setup**
1. Create EC2 instance for Jenkins agent
   - Instance type: t3.medium (minimum)
   - Storage: 50GB+ EBS
2. Configure AWS credentials:
   ```bash
   aws configure
   ```
3. Run the agent setup script:
   ```bash
   sh jenkins-agent.sh
   ```
4. Install required tools:
   - Docker
   - kubectl
   - helm
   - terraform
   - aws-cli

### 3. **Nexus Repository Setup**
1. Access Nexus UI at `http://<nexus-ip>:8081`
2. Create Maven repositories:
   - Create hosted repository named "backend"
   - Set version policy to "mixed"
   - Set layout policy to "permissive"
   - Allow redeployment
3. Configure Jenkins-Nexus integration:
   - Install "Nexus Artifact Uploader" plugin in Jenkins
   - Add Nexus credentials in Jenkins
   - Configure repository URLs
4. Create Docker repository:
   - Type: hosted
   - HTTP port: 8083
   - Enable Docker V1 API

### 4. **SonarQube Setup**
1. Launch SonarQube instance (t3.medium recommended)
   - Instance type: t3.medium
   - Storage: 30GB EBS
   - Security Group: Ports 22, 9000
2. Access SonarQube UI at `http://<sonarqube-ip>:9000`
3. Jenkins Integration:
   - Install SonarQube Scanner plugin
   - Configure SonarQube server in Jenkins
   - Add authentication token
   - Setup webhooks for analysis feedback
4. Configure Quality Gates:
   - Code Coverage: 80%
   - Duplicated Lines: 3%
   - Maintainability Rating: A
   - Security Rating: A
   - Reliability Rating: A

## 📊 **Monitoring and Security**

### **Monitoring Stack**
- **Metrics**
  - Prometheus for metrics collection
  - Grafana for visualization
  - Custom dashboards for:
    - Application metrics
    - Infrastructure metrics
    - Business metrics
- **Logging**
  - ELK Stack
  - Log rotation
  - Log aggregation
- **Alerting**
  - PagerDuty integration
  - Slack notifications
  - Email alerts

### **Security Measures**
- **Quality Gates**: 
  - Configured in SonarQube for code quality metrics
  - Branch protection rules
  - Required reviews
- **Security Scanning**: 
  - CodeQL analysis enabled
  - DAST scanning using Veracode
  - Container scanning
  - Dependency scanning
- **Monitoring**: 
  - Kubernetes metrics
  - Application performance monitoring
  - Infrastructure health checks
  - Custom metrics

## 🔐 **Security Best Practices**

### **Infrastructure Security**
- Bastion host for secure access
- Private subnets for sensitive resources
- IAM roles and policies
- Network security groups
- Regular security scanning
- Encrypted communication

### **Application Security**
- HTTPS everywhere
- WAF rules
- Rate limiting
- Input validation
- Output encoding
- CSRF protection
- XSS prevention

### **CI/CD Security**
- Secrets management
- Pipeline security
- Image scanning
- Dependency checking
- Compliance validation

---

## 📝 **Contributing**

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 **License**

This project is licensed under the MIT License - see the LICENSE file for details.

---

## ⭐ **Support & Contribution**  

If you find this repository helpful, consider:  
✅ Starring ⭐ the repository to support the project!  
✅ Forking 🍴 and contributing improvements or new installation guides  
✅ Reporting 🔥 issues or suggestions via GitHub Issues

---


