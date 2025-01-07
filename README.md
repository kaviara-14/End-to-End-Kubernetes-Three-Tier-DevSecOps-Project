# Automate Three-Tier Deployment with Terraform, Jenkins, and ArgoCD

## Overview
This project automates the deployment of a three-tier application using the following tools:

1. **Terraform**: Infrastructure provisioning for VPC, Subnets, and EKS clusters.
2. **Jenkins**: CI/CD pipelines for frontend and backend services.
3. **ArgoCD**: GitOps-based deployment for application services.
4. **Monitoring**: Prometheus and Grafana for monitoring the deployed environment.
   
---

## Steps for the deployment

### 1. Configure Jenkins Server 

- Create an ec2 instance with with instance type as **t2.2x large** and ami is **ubuntu Image** and volume as 30gb.
- In Key Pair section, choose proceed without a key pair we are going to use SSM(Session Manager).
- In the user data, provide the configurations you need to install.

```bash
# Install Java
sudo apt update -y
sudo apt install openjdk-17-jre -y
sudo apt install openjdk-17-jdk -y
java --version

# Install Jenkins
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y

# Install Docker
sudo apt update
sudo apt install docker.io -y
sudo usermod -aG docker jenkins
sudo usermod -aG docker ubuntu
sudo systemctl restart docker
sudo chmod 777 /var/run/docker.sock


# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install

# Install Kubectl
sudo apt update
sudo apt install curl -y
sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client


# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version

# Install Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y

# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy -y

# Run Docker Container of Sonarqube
docker run -d  --name sonar -p 9000:9000 sonarqube:lts-community

# Intall Helm
sudo snap install helm --classic

```
---

### 2. Terraform Deployment using Jenkins CI/CD

- Login to Jenkins Server,Go to Manage Jenkisn and install the required plugins -> store the sensitive information in credentials.
- Create a declarative pipeline and add your jenkinsFile code for creating aws infrastructure using terraform.
- Terraform scripts create the following, **VPC with both public and private subnets and EKS cluster deployed in private subnets**.
- Create a Jump server(ec2 instance) in the public subnet, so that we can access securely to our eks clusters in private subnet.

```bash
# set the kube config,using this we can connect to the aws eks cluster.
aws eks update-kubeconfig --region us-east-1 --name dev-medium-eks-cluster

# Download the IAM policy for the LoadBalancer.
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json

# create the IAM Policy
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json

# Create a Service Account
eksctl create iamserviceaccount \
      --cluster=dev-medium-eks-cluster \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name AmazonEKSLoadBalancerControllerRole \
      --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy \
      --approve \
      --region=us-east-1

# Deploy the AWS Load Balancer Controller
helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=my-cluster \
      --set serviceAccount.create=false \
      --set serviceAccount.name=aws-load-balancer-controller
```
 
![Screenshot 2025-01-06 162927](https://github.com/user-attachments/assets/6546bd52-ac62-4f86-b82b-bcbd3984d54e)
![Screenshot 2025-01-06 163454](https://github.com/user-attachments/assets/e82afded-809e-40a5-aa85-5a6fd49219b2)
![image](https://github.com/user-attachments/assets/eacdf1eb-db83-444c-9187-4fc720189398)

---

### 3. CI/CD Pipelines for Frontend Frontend

- Clean Up the workspace
- Clone the repository from github 
- Perform SonarQube analysis for code quality.Go to SonarQube Server -> Generate a token -> create and configure the webhook -> create a project in sonar-qube for frontend for code quality check -> under **Execute the Scanner** section you will get commands to run the sonarqube analysis for frontend.
- Conduct OWASP Dependency Check for vulnerabilities.
- Scan the filesystem using Trivy.
- Build the Docker image for the Frontend Service.
- Push the image to the AWS ECR Registry.
- Scan the Docker image using Trivy.
- Update the `frontend/deployment.yml` in GitHub with the new image.
     
![Screenshot 2025-01-06 163018](https://github.com/user-attachments/assets/68aeb6cb-dc2c-4346-881a-da7395be7f02)
![Screenshot 2025-01-06 162852](https://github.com/user-attachments/assets/8761061a-d0d1-45dc-8440-6681ceb825eb)
![Screenshot 2025-01-06 163432](https://github.com/user-attachments/assets/3d6eadc8-c81c-47fc-8141-11ac29145dad)

---

### 3. CI/CD Pipelines for Backend Service

- Clean Up the workspace.
- Clone the repository from github
- Perform SonarQube analysis for code quality. Go to SonarQube Server -> create a project in sonar-qube for backend to do a code quality check -> under **Execute the Scanner** section you will get commands to run the sonarqube analysis for backend.
- Conduct OWASP Dependency Check for vulnerabilities.
- Scan the filesystem with Trivy.
- Build the Docker image for the Backend Service.
- Push the image to the AWS ECR Registry.
- Scan the Docker image using Trivy.
- Update the `backend/deployment.yml` in GitHub with the new image.
     
![Screenshot 2025-01-06 163044](https://github.com/user-attachments/assets/d4f70d34-bb29-4fb2-b575-3fe110b73d1f)
![Screenshot 2025-01-06 162852](https://github.com/user-attachments/assets/8761061a-d0d1-45dc-8440-6681ceb825eb)
![Screenshot 2025-01-06 162914](https://github.com/user-attachments/assets/4e607913-82b1-4a43-899c-fc0b3cc9a375)

---

### 4. ArgoCD Deployment with AWS EKS

- Install the necessary tools in the Jump Server.
  
```bash
# Install ArgoCD
kubectl create namespace argocd 
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml

# By default, argocd-server is not publically exposed. In this scenario, we will use a Load Balancer to make it usable, to get the DNS.
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get the Load balancer DNS
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname'`
echo $ARGOCD_SERVER
export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`
echo $ARGO_PWD

#Name space Argocd application
kubectl create namespace three-tier
```

- Take the Load Balancer URL and paste it in the browser and login into the ArgoCD and configure and deploy the Application in this order one by one ,repo url and all others details will be same but path will different.
   - Name: three-tier-database and Path: Kubernetnes-Manifests-File/Database,
   - Name: three-tier-backend and Path: Kubernetnes-Manifests-File/Backend,
   - Name: three-tier-frontend and Path: Kubernetnes-Manifests-File/Frontend.
- In this Path we have respective yml files to create service and deployments.And once it is done check all the application is healthy or Not.

![Screenshot 2025-01-06 163343](https://github.com/user-attachments/assets/93f51372-a5d3-426a-9f27-043f30f2fd2b)
![Screenshot 2025-01-06 163245](https://github.com/user-attachments/assets/7ef08fb5-f525-4cdc-afb4-aa1653fbe29c)
![Screenshot 2025-01-06 163258](https://github.com/user-attachments/assets/f7759be3-8fd6-4f37-a415-e6a3aba450fc)
![Screenshot 2025-01-06 163315](https://github.com/user-attachments/assets/b217b560-7cb8-48a9-b60a-521ff1371026)

---
### 5. Route53 and deploying Ingress Controller

- Create Ingress Application in the ArgoCD and give path as **Kubernetnes-Manifests-File/** other details are common.Once your Ingress application is deployed. It will create an Application Load Balancer
You can check out the load balancer named with k8s-three.
- Now, Copy the ALB-DNS and go to your Route53 and register domain and go to DNS and add a CNAME type with hostname backend then add your ALB in the Answer and click on Save
- I have created a subdomain kaviarasu.study
- Now, hit your subdomain after 2 to 3 minutes in your browser to see the magic.

![Screenshot 2025-01-06 193143](https://github.com/user-attachments/assets/e5b73f6b-32b4-43b9-a914-7c71cbf30a74)

### 5. Monitoring with Prometheus and graffana
- **Tools Used**:
  - Prometheus for metrics collection.
  - Grafana for visualization and dashboarding.
- **Observability**:
  - Dashboards to monitor EKS cluster health and application performance.

![prometheus](https://github.com/user-attachments/assets/bfb08bab-9f5a-478c-8fd8-35460419f8d5)

--- 
