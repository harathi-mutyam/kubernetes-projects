# kubernetes-projects
## 1. KOPS-Based Kubernetes Cluster Deployment

Production-style Kubernetes cluster setup on AWS using KOPS.

**Features**
```shell
Kubernetes cluster provisioning using KOPS
Route53 integration
S3 bucket for KOPS state store
EC2 worker and master nodes
NGINX Ingress Controller
Application deployment on Kubernetes
```
**Technologies Used**
```shell
KOPS
Kubernetes
AWS EC2
Route53
S3
kubectl
NGINX Ingress
```
## 2. AWS EKS Deployment with Terraform & NGINX Ingress
Production-grade Amazon EKS deployment using Terraform and Kubernetes manifests.
**Features**
```shell
Private Amazon EKS cluster
Terraform Infrastructure as Code
Bastion host setup
AWS Network Load Balancer (NLB)
NGINX Ingress Controller
Route53 / GoDaddy DNS integration
Persistent storage using PVC
Secure microservices deployment
```
**Technologies Used**
```shell
Amazon EKS
Terraform
Kubernetes
AWS NLB
Route53
IAM

VPC
NAT Gateway
NGINX Ingress Controller
```
## 3. eks-acm-alb-production-cluster
### Production-Grade AWS EKS Deployment with ALB, TLS & Route53 path and host based (Terraform Automated)
```shell

1. AWS Load Balancer Controller (ALB)
2. HTTPS using AWS Certificate Manager (ACM)
3. Path-Based Routing
4. Host-Based Routing
5. Secure TLS termination at ALB
6. Microservices architecture (cart, product, payments)
7. Route53 / GoDaddy DNS integration
```
