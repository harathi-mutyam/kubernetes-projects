# Production-Grade AWS EKS Deployment(Terraform) Voting App 

## Prerequisites Setup for This Repository 
```shell
- AWS Load Balancer Controller (ALB)

- Install AWS CLI using Chocolatey  aws --version

- Install Terraform using Chocolatey  terraform -version
```
# Step 1: eks-voting-app – Terraform Deployment Guide

## 1. Clone the Repository 
### Locally using terminal
  git clone https://github.com/harathi-mutyam/kubernetes-projects.git

or 

### Using VS Code

**In vs code-->click on terminal-->new terminal-->select git bash-->Navigate to your desired local directory --> run git clone command**


```shell

git clone https://github.com/harathi-mutyam/kubernetes-projects.git

```
after cloning finished--->cick on file-->open Folder-->select your cloned repository(project folder)

```shell
ls
cd eks-voting-app

ls
cd terraform
ls
cd EKS
```
Make sure this folder contains: main.tf ,eks.tf,vpc.tf,provider.tf,dev.tfvars

## 2. Configure AWS CLI (mandatory):** for that go to browser create a IAM user (eks-user) in aws console for this project save access key and secret keys locally
```shell
aws configure

Set:

AWS Access Key : give your access key
Secret Key  : give your secret key
Region → us-east-1
json
```
## 3. Create S3 backend bucket (if not already created)

Through aws console or through command
```shell
aws s3 mb s3://your-terraform-state-bucket --region us-east-1
```

in **backend.tf** change the bucket name 

bucket       = "backend-bucket-final-6526"   #create s3 bucket through aws console .and replace bucket name here

Update the region based on your AWS region. in backend block 

### Make changes in dev.tfvars 

Change Ami id of Ubuntu Server Based on region

ami_id        = "ami-091138d0f0d41ff90"  #replace with your ubuntu ami id based on region

Instance type based on your project size

instance_type = "c7i-flex.large"  #change this to t3.small

aws_region                = "us-east-1"   #change region

instance_types   = ["c7i-flex.large"]  # t3.small

if reguired change 

kubernetes_version        = "1.34" also

## 4. Create EC2 Key Pair (VERY IMPORTANT) manually through aws console
## 5. Initialize Terraform
```shell
terraform init
```  
This will: download AWS provider ,initialize backend (S3) ,prepare modules

## 6. Validate configuration**
```shell
terraform validate
```
## 7. Plan infrastructure**
```shell
terraform plan -var-file="dev.tfvars"    #because all global configuration settings are defined in dev.tfvars. If you run only terraform plan, it will prompt you to enter values manually.”
```
We use dev.tfvars with terraform plan so Terraform already knows all values. If we don’t use it, Terraform will stop and ask us to enter them one by one.

## 8. Apply infrastructure**
```shell
terraform apply -var-file="dev.tfvars"

Type:yes
```
After resource creation completes, verify in the AWS Console that the bastion server, cluster, and VPCs ,IAM Roles are available. The process usually takes 10–15 minutes. Then copy the bastion server’s public IP address and launch a new Git Bash.

# Step 2: Post Deployment (Bastion Access & Kubernetes Setup)

### 1. Access Bastion Host
**Copy Bastion public IP from AWS Console**
**Open new Git Bash terminal**
**Connect using SSH:**

```shell
ssh -i Downloads/ec2_keypair.pem ubuntu@bastion_public_ip
```

### 2. Configure AWS
```shell
aws configure    #enter here access keys and secret keys ,region and json

AWS Access Key : give your saved access key

Secret Key  : give your saved secret key

Region → us-east-1

json
```
### 3. Configure Kubernetes access
```shell
aws eks update-kubeconfig --region us-east-1 --name dev-eks-demo
```
**Verify:**
```shell
kubectl get nodes
```
### 4. Install Helm (inside bastion)
```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## 5. Install AWS Load Balancer Controller
```shell
helm repo add eks https://aws.github.io/eks-charts
helm repo update
```

Install:

```shell
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=(dev-eks-demo) \
  --set region=us-east-1 \
  --set vpcId=(vpc-id) \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=(IAM-role-arn)
```

replace with your clustername ,vpc id and iam role of loadbalancer(search in aws console iam-->roles-->search  AWSLoadBalancerControllerRole   copy the arn ) arn and remove () also

## 6. Clone your repository on the Bastion Server for Deploying the Voting Application
Recommended Deployment Order

To ensure all application components start correctly and dependencies are available, deploy the Kubernetes resources in the following order.

```shell
git clone https://github.com/harathi-mutyam/kubernetes-projects.git
ls
cd kubernetes-projects
ls
cd eks-voting-app
ls
cd k8s-specifications
ls
```
### Step 1:  Create Namespace
Create a dedicated namespace for the voting application:
```shell
kubectl create namespace voting-app

```
```shell
 kubectl get ns

ls
```
**open  alb-ingress.yaml file replace host: vote.yourdomain.com** 
```shell
vim alb-ingress.yaml

```
save the file :wq!

**Note: create a Domain in Godaddy later** 


### Step 3: Deploy all Manifests in the following order

**Deploy PostgreSQL Database**

Deploy the PostgreSQL database first because the worker service depends on it.

```shell
 
kubectl apply -f db-deployment.yaml -n voting-app

kubectl apply -f db-service.yaml -n voting-app

```
**Deploy Redis**

Redis acts as the message queue between the voting application and the worker service.

```shell
kubectl apply -f redis-deployment.yaml -n voting-app

kubectl apply -f redisservice.yaml -n voting-app
```

```shell
 
 kubectl apply -f worker-deployment.yaml -n voting-app
```

**Deploy Voting Application**

Deploy the frontend voting application.

```shell
 kubectl apply -f vote-deployment.yaml -n voting-app

 kubectl apply -f vote-service.yaml -n voting-app
```

**Deploy Result Application**

Deploy the result application used to display voting statistics.

```shell
 kubectl apply -f result-deployment.yaml -n voting-app

   kubectl apply -f result-service.yaml -n voting-app

```
**Deploy Worker Service**
```shell
The worker transfers votes from Redis to PostgreSQL.

kubectl apply -f worker-deployment.yaml -n voting-app
```
**Deploy ALB Ingress**

Finally, expose the applications externally using AWS Application Load Balancer (ALB) Ingress.

```shell
 kubectl apply -f alb-ingress.yaml
```
## Step 4.Verify Deployment

**Check all pods:**
```shell

kubectl get pods -n voting-app
```

**Check services:**

```shell
kubectl get svc -n voting-app
```
**Check ingress:**
```shell

kubectl get ingress -n voting-app

```
**You will see:**

**ALB DNS name → k8s-default-xxxx.elb.amazonaws.com**

## Step 5: Connect Domain → ALB using Route 53

## You can use either either procedure 1((Recommended for AWS-native setup) or procedure 2(GoDaddy-managed DNS).

# PROCEDURE 1 (Recommended – Route 53 Managed DNS)

### **Step 1: Open Hosted Zone**

Now go to Route53--> Hosted Zone → ehmutyam.xyz

**Step 2: Create A Records (Alias → ALB)**

Create records: For EACH subdomain:
1. **cart.ehmutyam.xyz**

REcord Name: **cart **
Record Type: **A record**

Enable: Alias = **YES**

Route Traffic to : **select Alias to Application and Classic Load Balancer**

select region of your ALB : US East(N. Virginia)
select **your ALB DNS here** (for example: dualstack.k8s-ecommerc-ecommerc-949981ca5a-1099050560.us-east-1.elb.amazonaws.com)

2. **product.ehmutyam.xyz**

REcord name: product 

same foloow here also

3. **payments.ehmutyam.xyz**

REcord name: payments

same ALB


**Check it in browser :** with host names

https://cart.ehmutyam.xyz → output will be Welcome to cart Service    EnggVille

https:product.ehmutyam.xyz → Product Service

https:payments.ehmutyam.xyz → Payments Service

# PROCEDURE 2 (GoDaddy CNAME Method)
Use this ONLY if you are NOT using Route 53 NS delegation. 
## Step 1: Remove Route 53 NS Setup

```shell
Remove NS records of R53 in Godaddy.com ADD CNAME records

use default ns records of godaddy.com 

DELETE ALL NS records if you have added in the first process for R53 Ns Records

⚠️ No NS records needed at all.
```

## Step 2: Create CNAME records in godaddy.com for https purpose 

DNS Management → Add Record

```shell
type: CNAME       Name: cart          Data Value: alb DNS name 

type: CNAME       Name: product       Data Value: alb DNS name 

type: CNAME       Name: payments      Data Value: alb DNS name 
```
Example:

k8s-ecommerc-xxxxx.us-east-1.elb.amazonaws.com




A simple distributed application running across multiple Docker containers.

## Getting started

Download [Docker Desktop](https://www.docker.com/products/docker-desktop) for Mac or Windows. [Docker Compose](https://docs.docker.com/compose) will be automatically installed. On Linux, make sure you have the latest version of [Compose](https://docs.docker.com/compose/install/).

This solution uses Python, Node.js, .NET, with Redis for messaging and Postgres for storage.

Run in this directory to build and run the app:

```shell
docker compose up
```

The `vote` app will be running at [http://localhost:8080](http://localhost:8080), and the `results` will be at [http://localhost:8081](http://localhost:8081).

Alternately, if you want to run it on a [Docker Swarm](https://docs.docker.com/engine/swarm/), first make sure you have a swarm. If you don't, run:

```shell
docker swarm init
```

Once you have your swarm, in this directory run:

```shell
docker stack deploy --compose-file docker-stack.yml vote
```

## Run the app in Kubernetes

The folder k8s-specifications contains the YAML specifications of the Voting App's services.

Run the following command to create the deployments and services. Note it will create these resources in your current namespace (`default` if you haven't changed it.)

```shell
kubectl create -f k8s-specifications/
```

The `vote` web app is then available on port 31000 on each host of the cluster, the `result` web app is available on port 31001.

To remove them, run:

```shell
kubectl delete -f k8s-specifications/
```

## Architecture

![Architecture diagram](architecture.excalidraw.png)

* A front-end web app in [Python](/vote) which lets you vote between two options
* A [Redis](https://hub.docker.com/_/redis/) which collects new votes
* A [.NET](/worker/) worker which consumes votes and stores them in…
* A [Postgres](https://hub.docker.com/_/postgres/) database backed by a Docker volume
* A [Node.js](/result) web app which shows the results of the voting in real time

## Notes

The voting application only accepts one vote per client browser. It does not register additional votes if a vote has already been submitted from a client.

This isn't an example of a properly architected perfectly designed distributed app... it's just a simple
example of the various types of pieces and languages you might see (queues, persistent data, etc), and how to
deal with them in Docker at a basic level.
