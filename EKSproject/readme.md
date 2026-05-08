# Production-Grade AWS EKS Deployment with Terraform & Kubernetes (vProfile Project)

## 1. Project Overview

This project demonstrates a production-ready microservices deployment on AWS using:
```shell
Terraform (Infrastructure as Code)
Amazon EKS (Kubernetes Cluster)
NGINX Ingress Controller
AWS Network Load Balancer (NLB)
Route53 / GoDaddy DNS
Bastion Host for secure access
```
## 2.Architecture Overview
```shell
User
  ↓
DNS (Route53 / GoDaddy)
  ↓
AWS Network Load Balancer (NLB)
  ↓
NGINX Ingress Controller
  ↓
Kubernetes Services
  ↓
Pods (App, DB, Cache, RabbitMQ)
```

The entire infrastructure layer is automated using Terraform.

## 3.AWS Infrastructure (Terraform)
```shell
Components created:
VPC (public + private subnets)
Internet Gateway (IGW)
NAT Gateway
Route Tables
Security Groups
EKS Cluster (private)
Managed Node Groups
IAM Roles + OIDC + IRSA
```
## 4.Prerequisites

Before starting the project, ensure the following tools and configurations are completed:
```shell
AWS Account
Terraform Installed
AWS CLI Installed
Git Installed
VS Code / Git Bash
```
## **Steps to Clone and Run the Project in VS code**

### **1. Create a Local Working Directory (Optional)** 
Create a folder on your local machine to organize the project files.
```shell
mkdir kubernetes-projects
cd kubernetes-projects
```
This step is optional but recommended for better project structure and management.

### **2. Clone the Repository**

Open VS Code Terminal or Git Bash and run:
```shell
git clone https://github.com/harathi-mutyam/kubernetes-projects.git
```
In Vs code -->File-->open folder-->select Your Project folder

### **3. After Cloning the Repository**

Make the following changes:

Update the region in dev.tfvars based on the location where you want to create your infrastructure (such as EKS, VPC, etc.).

Example:

region = "us-east-1"

Ensure the region in backend.tf matches the region where your Terraform state storage (for example, an S3 bucket) is hosted.

### Configure AWS CLI (Mandatory)

Go to browser

To interact with AWS services from your local machine or bastion host, configure AWS CLI using IAM user credentials.

**Create IAM User in AWS Console**

Login to AWS Console as Root User.
```shell
AWS Console → IAM → Users → Create User 
Create a user (example):

eks-user

Attach required permissions:

AdministratorAccess (for learning/demo projects)

After user creation:
Download or save:
AWS Access Key
AWS Secret Access Key
```
**In VS code**
```shell
aws --version    #check aws version
aws configure

Set:

AWS Access Key : give your access key
Secret Key  : give your secret key
Region → us-east-1
json

```

### 4. Create s3 bucket in your region through command or manually
```shell
#create a bucket
 aws s3 mb s3://eksprojectstatebkt8526 --region us-east-1

 aws s3api put-bucket-versioning --bucket eksprojectstatebkt8526 --versioning-configuration Status=Enabled

#enable versioning also
```
 change bucketname and region in backend.tf 
 
 **Create EC2 Key Pair in AWS console manually update it in dev.tfvars**
 
### **5. Navigate to the Terraform Directory**

Always run Terraform commands from the folder where main.tf exists:
```shell
cd kubernetes-projects
ls
cd EKSproject
cd terraform
cd EKS
```
### **6. Verify Terraform Installation**
```shell
terraform version
```
if not installed install terraform through choletty

**7. Initialize Terraform**
```shell
terraform init
```
**validate the terraform code**
```shell
terraform validate
```
**7. Plan the Infrastructure**
```shell
terraform plan -var-file="dev.tfvars"
```
**8. Apply the Changes**
```shell
terraform apply -var-file="dev.tfvars"
```
After resource creation completes, verify in the AWS Console that the bastion server, cluster, and VPCs ,IAM Roles are available. 

The process usually takes 10–15 minutes. Then copy the bastion server’s public IP address and launch a new Git Bash session.

# Post-Provisioning Setup (Inside Bastion Host)
After Terraform completes:

# AWS SETUP
## Step 1: Login to Bastion EC2 Instance through gitbash

From your laptop terminal:

```shell
ssh -i Downloads/kopskey.pem ubuntu@PUBLIC-IP
```
🚀 PHASE 3 — INSTALL REQUIRED TOOLS
## Step 8: Install AWS CLI
```shel
snap install aws-cli --classic
```
Check:
```shell
aws --version
```
## Step 9: Configure AWS access keys and secret keys
```shell
aws configure

Enter:

Access Key  : paste your access key

Secret Key :  paste your secret key

Region: us-east-1

Output:  json
```
## Step 10:Connect kubectl to EKS Cluster
```shell
aws eks update-kubeconfig --region us-east-1 --name dev-eks-demo
#Verify cluster connection
kubectl get nodes
```
## Step 11: Install NGINX Ingress Controller on EKS
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```
## Expose Ingress Controller using AWS Load Balancer
This command tells AWS:“Create a public (internet-facing) Network Load Balancer (NLB) for this service”
```shell
kubectl annotate svc ingress-nginx-controller -n ingress-nginx service.beta.kubernetes.io/aws-load-balancer-scheme=internet-facing --overwrite
```
## Step 11:  Clone Project from GitHub
```shell
git clone https://github.com/harathi-mutyam/kubernetes-projects.git
```
#### Navigate into project
```shell
cd kubernetes-projects
ls
cd EKSproject
cd k8s
ls
```
#### Check PVC file
```shell
cat dbpvc.yaml
```
#### Update Storage Class

```shell
vim dbpvc.yaml
```
Change  in yaml file storageClassName: gp2  
#### Update Ingress file

```shell
vim ingress.yaml
```
Replace in yaml file 
hostname: vprofile.ehmutyam.xyz   (replace with your Domain Name)

Ensure ingress class must be nginx

ingressClassName: nginx

#### first apply Storage (PVC)
```shell
kubectl apply -f dbpvc.yaml
```
Database (MySQL) needs storage

PVC must exist before Pod starts

Otherwise MySQL pod will fail or stay in Pending

#### Then Apply all other manifests
```shell
kubectl apply -f .
```
#### Check Ingress status

```shell
kubectl get ingress
NAME           CLASS   HOSTS                   ADDRESS   PORTS
vpro-ingress   nginx   vprofile.ehmutyam.xyz  <pending>
```
wait 3–5 mins to update the address .ADDRESS → AWS NLB DNS
```shell
kubectl get ingress
NAME           CLASS   HOSTS                   ADDRESS                                                                         PORTS   AGE
vpro-ingress   nginx   vprofile.ehmutyam.xyz   a5463fd475a8a449eb4b4efe54f62e61-680ccb73d736915e.elb.us-east-1.amazonaws.com   80      7m47s
```
#### Verify DNS and Access using these commands

```shell
nslookup vprofile.ehmutyam.xyz

curl http://vprofile.ehmutyam.xyz

dig vprofile.ehmutyam.xyz

```
## Step 12: DNS Configuration Approaches

  ### 1. **Application Access Architecture**
     ```shell
      vprofile.ehmutyam.xyz
              ↓
      Route53 / GoDaddy DNS
              ↓
      AWS Network Load Balancer (NLB)
              ↓
      NGINX Ingress Controller
              ↓
      Kubernetes Services
              ↓
      Pods (App, DB, Cache, MQ)
     ```

### **2. DNS Configuration Approaches**
## Approach 1: Route53 (Recommended for AWS-native setups)
#####  Step 1: Create Hosted Zone
    ```shell
      Go to Route53 → Hosted Zones
      Domain: ehmutyam.xyz
      Type: Public Hosted Zone
    ```shell
#####  Step 2: Copy Nameservers
AWS provides 4 NS records:
```shell
ns-xxx.awsdns-xx.net
ns-xxx.awsdns-xx.org
ns-xxx.awsdns-xx.com
ns-xxx.awsdns-xx.co.uk
```

##### Step 3: Update GoDaddy Nameservers
```shell
Replace GoDaddy default DNS with R53 NS values
Paste Route53 nameservers
Wait 5–20 minutes for propagation
```

#### Step 4: Create CNAME Record in Route53
```shell
Field	        Value
Record name	  vprofile
Type	        CNAME
Value	        AWS NLB DNS
TTL         	300
```
### Step 5: Access Application
  http://vprofile.ehmutyam.xyz
  

## Approach 2: Direct GoDaddy DNS (Simpler & common in projects)
### Step 1: Keep GoDaddy DNS
      No nameserver change required

### Step 2: Get NLB DNS from Gitbash
```shell
    kubectl get svc -n ingress-nginx
```
Copy:
```shell
xxxx.elb.us-east-1.amazonaws.com
```
### Step 3: Create CNAME in GoDaddy
```shell
Field	   Value
Type	   CNAME
Name	   vprofile
Value	   NLB DNS
TTL	     1 Hour
```
### Step 4: Access Application
  http://vprofile.ehmutyam.xyz



##### Debugging Commands optional

```shell
#Check ingress details
kubectl describe ingress vpro-ingress
#Check all services
kubectl get svc -A
#Check ingress controller service
kubectl get svc -n ingress-nginx
```
## Step 12: Deletion Process
```shell
kubectl delete svc ingress-nginx-controller -n ingress-nginx

kubectl delete pvc db-pv-claim

kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml

kubectl delete -f .
```

## Step 13: 
from VS Code deletes all infrastructure created by Terraform
```shell
terraform destroy -var-file="dev.tfvars"
```


