# **Production-Grade AWS EKS Deployment with NGINX, Route53 (Terraform Automated)**

## 📌 Overview

This project provisions a production-ready private Amazon EKS cluster using Infrastructure as Code (Terraform) and deploys microservices using Kubernetes with:

- AWS Nginx network load Controller

- Kubernetes Ingress

- Route53 DNS configuration

- Bastion host for secure access

The entire infrastructure layer is automated using Terraform.

## Architecture
Core AWS Services Used
- Amazon Web Services
- Amazon EKS
- AWS Network Load Balancer
- Amazon Route 53


## Infrastructure Provisioned with Terraform
### Networking (Custom VPC)
- VPC
- Public Subnets
- Private Subnets
- Internet Gateway (IGW)
- NAT Gateway
- Route Tables
- Security Groups

#### Architecture model:
```
Public Subnet:
  - Bastion Host
  - NAT Gateway
  - ALB

Private Subnet:
  - EKS Worker Nodes
```

### Private EKS Cluster
- Private Endpoint Enabled
- IAM Roles & Policies
- OIDC Provider
- IRSA (IAM Roles for Service Accounts)
- Managed Node Groups

#### Security-first design:
- No public access to worker nodes
- Access only via Bastion host

### Bastion Host
Used for:
- Secure SSH access
- Kubectl access to private cluster
- Controller installation

**Steps to Clone and Run the Project**

**1. Create a Local Folder**

Create a folder in your local directory named production-eks.

**2. Clone the Repository**

Open VS Code (or Git Bash) and clone the repository:

git clone https://github.com/harathi-mutyam/kubernetes-projects.git

**3. After Cloning the Repository**

Make the following changes:

Update the region in dev.tfvars based on the location where you want to create your infrastructure (such as EKS, VPC, etc.).

Example:

region = "us-east-1"

Ensure the region in backend.tf matches the region where your Terraform state storage (for example, an S3 bucket) is hosted.
Create s3 bucket in your region through command or manually

 aws s3 mb s3://eksprojectstatebkt8526 --region us-east-1
 enable versioning also

 change bucketname and region in backend.tf 
 
**4. Navigate to the Terraform Directory**

Always run Terraform commands from the folder where main.tf exists:
```shell
cd kubernetes-projects
ls
cd EKSproject
cd terraform
cd EKS
```
**5. Verify Terraform Installation**

terraform version

**6. Initialize Terraform**

terraform init

**validate the terraform code**

terraform validate

**7. Plan the Infrastructure**

terraform plan -var-file="dev.tfvars"

**8. Apply the Changes**

terraform apply -var-file="dev.tfvars"


## Post-Provisioning Setup (Inside Bastion Host)
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


