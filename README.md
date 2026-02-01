# three-tier-eks-sample-project
## Overview:

- A sample project to deploy a 3-tier app by building Docker images for frontend and backend, pushing them to a registry, and then deploying the stack on Kubernetes by provisioning an AWS EKS cluster.

## Step - 1: Prerequisites

1. Make sure you have the following installed on your VM:
   - AWS CLI v2 
   - Docker
   - kubectl
   - Helm (for chart deployment)
   - Terraform

## Step - 2: AWS IAM & AWS CLI Setup

1. Create an IAM user with AdministratorAccess (or least privileges for EKS, IAM, EC2, ECR).
2. Configure AWS CLI (aws configure)

## Step - 3: Create S3 Bucket, DynamoDB Table
1. S3 Bucket
   
    ```bash
      aws s3 mb s3://<your-unique-bucket-name>
    ```
2. DynamoDB Table (for state locking) - Replace region

    ```bash
      aws dynamodb create-table \
    --table-name eks-terraform-lock \
    --attribute-definitions AttributeName=LockID,AttributeType=S \
    --key-schema AttributeName=LockID,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region us-east-1
    ```
3. Edit Backend Configuration
   
    ```bash
    ls terraform
    vi backend.tf
    ```
   - Replace the bucket name and region with yours. 
   - Add DynamoDB table name to the file: dynamodb_table = "your-table-name" and save the file.
   

## Step - 4: Provision EKS Cluster with Terraform

1. Clone the repo into your VM and go to the terraform folder:
2. Initialize Terraform
  
     ```bash
     cd terraform
     terraform init
     ```
3. Plan the infrastructure

     ```bash
     terraform plan
     ```
4. Create resources - Terraform will provision VPC, subnets, Node groups, IAM roles, security groups, etc.

     ```bash
     terraform apply --auto-approve
     ```
5. After creation, update your kubeconfig and verify:

     ```bash
     aws eks update-kubeconfig --name <your-cluster-name> --region <your-region>
     kubectl get nodes
     kubectl get pods -n kube-system
     ```
## Step - 5: Create ECR Repository

1. ECR repository for frontend:

   ```bash
   aws ecr create-repository \
   --repository-name workshop-frontend \
   --region ap-southeast-2
   ```
2. ECR repository for frontend:

   ```bash
   aws ecr create-repository \
   --repository-name workshop-backend \
   --region ap-southeast-2
   ```
3. Authenticate Docker to ECR:
   - ECR is a private registry by default. So only authenticated AWS principals can push/pull images to/from it.
   - AWS provides a temporary login token that Docker can use as a password to access the ECR.
   - EKS worker nodes can pull from ECR because they have IAM roles, and the IAM role allows ECR access. 

   ```bash
   aws ecr get-login-password --region ap-southeast-2 \
   | docker login \
     --username AWS \
     --password-stdin <YOUR_ACCOUNT_ID>.dkr.ecr.ap-southeast-2.amazonaws.com
   ```
  
 ## Step - 6: Build & Push Docker Images
 
 1. Go to the app directory and then to the frontend folder where your Dockerfile for frontend exists. Build the image for your frontend:

    ```bash
    docker build -t workshop-frontend:v1 .
    docker tag workshop-frontend:v1 \
    <YOUR_ACCOUNT_ID>.dkr.ecr.ap-southeast-2.amazonaws.com/workshop-frontend:v1
    docker push <YOUR_ACCOUNT_ID>.dkr.ecr.ap-southeast-2.amazonaws.com/workshop-frontend:v1
    ```
 2. Now move to the backend folder and do the same:

    ```bash
    docker build -t workshop-backend:v1 . 
    docker tag workshop-backend:v1 \
    <YOUR_ACCOUNT_ID>.dkr.ecr.ap-southeast-2.amazonaws.com/workshop-backend:v1
    docker push <YOUR_ACCOUNT_ID>.dkr.ecr.ap-southeast-2.amazonaws.com/workshop-backend:v1
    ```

## Step - 7: Create Kubernetes Namespace & Set Context

1. A namespace is a logical isolation inside a cluster. We create it to isolate the application (your entire 3-tier app is contained in a single namespace).
2. We have frontend and backend deployment, MongoDB, services, load balancer. If you don't create a namespace, everything goes into the default and can get messy.
3. And we set the context to avoid typing -n workshop repeatedly.

   ```bash
   kubectl create ns workshop
   kubectl config set-context --current --namespace=workshop
   ```

## Step - 8: Deploy Kubernetes Manifests

1. K8s manifests are in the k8s_manifests folder.
   - MongoDB:

     ```bash
     cd k8s_manifests/mongo
     kubectl apply -f secrets.yaml
     kubectl apply -f deploy.yaml
     kubectl apply -f service.yaml
     ```
   - Backend API:

     ```bash
     cd k8s_manifestsap
     kubectl apply -f backend-deployment.yaml
     kubectl apply -f backend-service.yaml
     ```
   - Frontend App:

     ```bash
     cd k8s_manifestsap
     kubectl apply -f frontend-deployment.yaml
     kubectl apply -f frontend-service.yaml
     ```
   - Expose Full Stack (Load Balancer):

     ```bash
     kubectl apply -f full_stack_lb.yaml
     ```
## Step - 8: Monitoring tools setup

1. Install Prometheus & Grafana:
   - Add Helm Repo

     ```bash
     helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
     helm repo update
     ```
   - Create Monitoring Namespace:

     ```bash
     kubectl create ns monitoring
     ```
   - Install kube-prometheus-stack: This installs Prometheus, Grafana, Alertmanager, Node Exporter, kube-state-metrics.

     ```bash
     helm install monitoring prometheus-community/kube-prometheus-stack \
     --namespace monitoring
     ```
   - Check its status by running:

     ```bash
     kubectl get pods -n monitoring
     ```
   - Expose Grafana:
     
     ```bash
     kubectl edit svc monitoring-grafana -n monitoring
     ```
     Change ClusterIP to LoadBalancer. Save & exit. Then,

     ```bash
     kubectl get svc -n monitoring
     ```
   - Accessing Grafana UI:
     The above command gives you services and you can see an external IP provided for grafa to access its UI. Access it through the browser using http://<your-external-IP>
     user name: admin
     password: access your password using below command,
     
     ```bash
     kubectl get secret monitoring-grafana \
     -n monitoring \
     -o jsonpath="{.data.admin-password}" | base64 -d
     ```
   - Importing Dashboards:
     Go to dashboards and click on import (+ symbol on top right) -> Enter dashboard ID -> Click on load -> Click on import


     

     
     


     
   

     
