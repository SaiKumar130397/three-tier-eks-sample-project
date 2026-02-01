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

## Step - 3: Provision EKS Cluster with Terraform

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
        ```
     
  
   

     
