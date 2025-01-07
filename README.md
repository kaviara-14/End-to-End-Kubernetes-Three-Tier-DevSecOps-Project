# Automate Three-Tier Deployment with Terraform, Jenkins, and ArgoCD

## Overview
This project automates the deployment of a three-tier application using the following tools:

1. **Terraform**: Infrastructure provisioning for VPC, Subnets, and EKS clusters.
2. **Jenkins**: CI/CD pipelines for frontend and backend services.
3. **ArgoCD**: GitOps-based deployment for application services.
4. **Monitoring**: Prometheus and Grafana for monitoring the deployed environment.
   
---

## Steps for the deployment

### 1. Terraform Deployment using Jenkins CI/CD
- **Infrastructure Setup**: Terraform scripts create the following:
  - VPC with both public and private subnets.
  - EKS cluster deployed in private subnets.
  - Jump server in the public subnet to access EKS in private subnets.
  - **Jenkins Server Deployment**: Automate provisioning of the Jenkins server using Terraform.
- Deployment is triggered and managed through Jenkins.
  
![Screenshot 2025-01-06 162927](https://github.com/user-attachments/assets/6546bd52-ac62-4f86-b82b-bcbd3984d54e)
![Screenshot 2025-01-06 163454](https://github.com/user-attachments/assets/e82afded-809e-40a5-aa85-5a6fd49219b2)
![Screenshot 2025-01-06 163432](https://github.com/user-attachments/assets/3d6eadc8-c81c-47fc-8141-11ac29145dad)

---

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
     
![Screenshot 2025-01-06 163018](https://github.com/user-attachments/assets/68aeb6cb-dc2c-4346-881a-da7395be7f02)

#### Backend Pipeline
- **Steps**:
  1. Perform SonarQube analysis for code quality.
  2. Conduct OWASP Dependency Check for vulnerabilities.
  3. Scan the filesystem with Trivy.
  4. Build the Docker image for the backend service.
  5. Push the image to the AWS ECR registry.
  6. Scan the Docker image using Trivy.
  7. Update the `backend/deployment.yml` in GitHub with the new image.
     
![Screenshot 2025-01-06 163044](https://github.com/user-attachments/assets/d4f70d34-bb29-4fb2-b575-3fe110b73d1f)

![Screenshot 2025-01-06 162852](https://github.com/user-attachments/assets/8761061a-d0d1-45dc-8440-6681ceb825eb)
![Screenshot 2025-01-06 162914](https://github.com/user-attachments/assets/4e607913-82b1-4a43-899c-fc0b3cc9a375)

---

### 3. ArgoCD Deployment
- **Services Deployed**:
  - Frontend
  - Backend
  - Database
  - **Ingress Configuration**: Route traffic to respective services using custom subdomains.
- **Data Persistence**:
  - Configure Persistent Volumes (PVs) and Persistent Volume Claims (PVCs) for database pods to ensure reliable data storage.
    
![Screenshot 2025-01-06 163343](https://github.com/user-attachments/assets/93f51372-a5d3-426a-9f27-043f30f2fd2b)
![Screenshot 2025-01-06 163245](https://github.com/user-attachments/assets/7ef08fb5-f525-4cdc-afb4-aa1653fbe29c)
![Screenshot 2025-01-06 163258](https://github.com/user-attachments/assets/f7759be3-8fd6-4f37-a415-e6a3aba450fc)
![Screenshot 2025-01-06 163315](https://github.com/user-attachments/assets/b217b560-7cb8-48a9-b60a-521ff1371026)

---

### 4. Monitoring
- **Tools Used**:
  - Prometheus for metrics collection.
  - Grafana for visualization and dashboarding.
- **Observability**:
  - Dashboards to monitor EKS cluster health and application performance.

![prometheus](https://github.com/user-attachments/assets/bfb08bab-9f5a-478c-8fd8-35460419f8d5)
![Screenshot 2025-01-06 193143](https://github.com/user-attachments/assets/e5b73f6b-32b4-43b9-a914-7c71cbf30a74)

--- 
