# Automate Three-Tier Deployment with Terraform, Jenkins, and ArgoCD

## Overview
This project automates the deployment of a three-tier application using the following tools:

1. **Terraform**: Infrastructure provisioning for VPC, Subnets, and EKS clusters.
2. **Jenkins**: CI/CD pipelines for frontend and backend services.
3. **ArgoCD**: GitOps-based deployment for application services.
4. **Monitoring**: Prometheus and Grafana for monitoring the deployed environment.
   
---

## Steps

### 1. Terraform Deployment using Jenkins CI/CD
- **Infrastructure Setup**: Terraform scripts create the following:
  - VPC with both public and private subnets.
  - EKS cluster deployed in private subnets.
  - Jump server in the public subnet to access EKS in private subnets.
  - **Jenkins Server Deployment**: Automate provisioning of the Jenkins server using Terraform.
- Deployment is triggered and managed through Jenkins.

### 2. CI/CD Pipelines for Frontend and Backend
#### Frontend Pipeline
- **Steps**:
  1. Perform SonarQube analysis for code quality.
  2. Conduct OWASP Dependency Check for vulnerabilities.
  3. Scan the filesystem with Trivy.
  4. Build the Docker image for the frontend service.
  5. Push the image to the AWS ECR registry.
  6. Scan the Docker image using Trivy.
  7. Update the `frontend/deployment.yml` in GitHub with the new image.

#### Backend Pipeline
- **Steps**:
  1. Perform SonarQube analysis for code quality.
  2. Conduct OWASP Dependency Check for vulnerabilities.
  3. Scan the filesystem with Trivy.
  4. Build the Docker image for the backend service.
  5. Push the image to the AWS ECR registry.
  6. Scan the Docker image using Trivy.
  7. Update the `backend/deployment.yml` in GitHub with the new image.

### 3. ArgoCD Deployment
- **Services Deployed**:
  - Frontend
  - Backend
  - Database
  - **Ingress Configuration**: Route traffic to respective services using custom subdomains.
- **Data Persistence**:
  - Configure Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) for database pods to ensure reliable data storage.

### 4. Monitoring
- **Tools Used**:
  - Prometheus for metrics collection.
  - Grafana for visualization and dashboarding.
- **Observability**:
  - Dashboards to monitor EKS cluster health and application performance.


