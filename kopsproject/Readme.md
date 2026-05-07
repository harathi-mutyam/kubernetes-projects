
# Prerequisites for Kops:
## Step 0.	Domain for Kubernetes DNS records.
      Login to your GoDaddy account and purchase a domain for Kubernetes.
          •	Ex. ehmutyam.xyz (yourdomain.com) from GoDaddy


# AWS SETUP
## Step 1: Login to AWS Console

Login with:

Root User
## Step 2: Create IAM User for Kops

### Kops is a third-party tool, to create infrastructure on AWS kops tool require permissions.For Kops create an IAM user with administrator access.

### Create Access keys for the kops-user for authentication.

Go to:

AWS Console → IAM → Users → Create User

Create user:

Username: **kops**

Click:   **Next**

Choose:  **Attach policies directly**

Select:  **AdministratorAccess**

Next-->  Create User
## Step 3: Create Access Key

Open created user:

IAM → Users → kops

Go to:  Security Credentials Tab -->  Click:   ---> **Create Access Key**

Choose:  **CLI** 

Select i understand check box

Download CSV file.  

Save: 
**Access Key**

**Secret Key**
locally in your laptop

 🚀 PHASE 2 — CREATE EC2 SERVER FOR KOPS

## Step 4: Launch EC2 Instance

### This instance is used for installing kops and kubectl, using which we can create our infrastructure on AWS.

Go to:

EC2 → Launch Instance

Settings: Setting	Value 

Name:	**kops**   

OS	:**Ubuntu**  

Type:	**c7i-flex.large**   

Storage: **20GB**


## Step 5: Create Security Group

Create:

Type: **SSH**	    Source: **Anywhere**

Name: kops-sg

## Step 6: Create Key Pair

Create: **kopskey.pem**

Download it.

Launch instance.

## Step 7: Login to EC2 Instance through gitbash

From your laptop terminal:

```shell
ssh -i Downloads/kopskey.pem ubuntu@<PUBLIC-IP>
```
**Become root user:**
```shell
sudo -i
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

Secret Key :  paste your access key

Region: us-east-1

Output:  json
```
## Step 10: Generate SSH Keys
```shell
ssh-keygen

Press Enter 4 times.
```
Check keys:
```shell
ls ~/.ssh/
```
## Step 11: Install Kops
```shell
curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops
mv kops /usr/local/bin/
```
Check:
```shell
kops version
```
## Step 12: Install kubectl
```shell
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Verify:
```shell
kubectl version --client
```
🚀 PHASE 4 — CREATE S3 BUCKET

## Step 13: Create S3 Bucket
```shell
aws s3api create-bucket \
--bucket kopsstatebkt7526 \
--region us-east-1
```
## Step 14: Enable Versioning
```shell
aws s3api put-bucket-versioning \
--bucket kopsstatebkt7526 \
--versioning-configuration Status=Enabled
```
Purpose: 

Kops stores cluster configuration in S3 bucket

🚀 PHASE 5 — CREATE DOMAIN & DNS

## Step 15: Create Hosted Zone

Go to:

AWS → Route53 → Hosted Zones

Create domain:

kopsvprofile.yourdomain.com   (Example: **kopsvprofile.ehmutyam.xyz**)

Choose:  **Public Hosted Zone**

The hosted zone creates 2 records – NS record and SOA record.

The NS record has four name servers, copy all the name servers and create records in GoDaddy.

## Step 16: Update NS Records in GoDaddy DNS

Login to your GoDaddy account and create 4 NS records for the name servers of the AWS Hosted Zone.

Copy 4 NS records from Route53.

Go to GoDaddy:   Domains → DNS   -->  Add all 4 NS records.

Wait:

5–10 minutes

🚀 PHASE 6 — CREATE KUBERNETES CLUSTER

## Step 17: Create kops cluster from kops ec2 instance:
```shell
kops create cluster \
--name=kopsvprofile.ehmutyam.xyz \
--state=s3://kopsstatebkt7526 \
--zones=us-east-1a,us-east-1b \
--node-count=2 \
--node-size=c7i-flex.large \
--control-plane-size=c7i-flex.large \
--dns-zone=kopsvprofile.ehmutyam.xyz \
--node-volume-size=12 \
--control-plane-volume-size=12 \
--ssh-public-key ~/.ssh/id_rsa.pub
```


**replace s3 bucket name with your bucket name**   

change node-size and control-plane-size from **t3.smal to lc7i-flex.large** 

**Replace id_rsa.pub** with your .pub key (we will know the key after running this command **ls ~/.ssh/** )
 
## Step 18: Apply Cluster
```shell
kops update cluster \
--name kopsvprofile.ehmutyam.xyz \
--state=s3://kopsstatebkt7526 \
--yes --admin
```

Replace kopsvprofile.ehmutyam.xyz with your kubeprofile.yourdomain.com

⏳ **Wait 10–15 minutes.**


## Step 19: Validate Cluster
```shell
kops validate cluster \
--name=kopsvprofile.ehmutyam.xyz \
--state=s3://kopsstatebkt7526
```
**Check nodes:**
```shell
kubectl get nodes
```
Expected:

1 master/control plane

2 worker nodes

🚀 PHASE 7 — DEPLOY vprokube APPLICATION

When kops create the cluster, it creates the kube config file which has all the information of the cluster.

#### optional commands to check the config

```shell
ls
ls .kube/
cat .kube/config
kubectl get nodes
```
## Step 20: Clone Repository
```shell
git clone https://github.com/harathi-mutyam/kubernetes-projects.git

```
Check files:
```shell
ls

```


## Step 21: Create an Ingress Controller:

Create an ingress controller with command
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
```
List the namespace and get the pods related to ingress namespace 
```shell
kubectl get ns
kubectl get pods -n ingress-nginx
or
kubectl get pods --namespace ingress-nginx
```
Check the status of the nodes 
```shell
kubectl get nodes
```
**Create an EBS volume for DB deployment**:

Changed to the kopsproject folder,changed to k8smanifests,  inside the repository and observe an EBS volume with dbpvc.yaml definition file.  
```shell

cd kopsproject

ls
cd kubedefs
cd k8smanifests
cat dbpvc.yaml
```
**create an EBS volume with dbpvc.yaml**
```shell
kubectl apply -f dbpvc.yaml    #PersistentVolumeClain/db-pv-claim created
```
**Check for the new EBS volume created for DB volume persistent Claim in your AWS EC2 console**

## Step 22: Deploy Application

If manifests are inside kubedefs/:

```shell
kubectl apply -f .
```
## Step 23: Verify Pods
```shell
kubectl get pods
```
Pods should become:

Running

## Step 24: Check Services
```shell
kubectl get svc
kubectl get deploy
```
Check PVC and storage class of the Volume
```shell
kubectl get pvc
kubectl get sc
```
Check the hostname and address mapping for the ingress controller.  
```shell
kubectl get ingress
kubectl describe ingress vpro-ingress
```
## Create a CNAME record in GoDaddy hostname mapped to the Load Balancer DNS name created by the ingress controller.

go to go daddy.com  --> Click on your Domain (ehmutyam.xyz) --> Domain --> DNS --> Add New Record 
```shell
Type: CNAME
Name: kopsvprofile
Value : Loadbalancer DNS Name Paste here
Save the Record
```
## Step 25: Access Application

**Check with the DNS name of the Load Balancer in Browser**, you will get 404 for nginx controller

Ingress Controller will only forward request to the hostname mapped in the domain registrar.

Application cannot be accessed using Load Balancer endpoint, can be accessed only with the hostname. 

Login and check the user list, click on any user the data will be inserted into cache. 

If you go back and click on the user again, the data will be from cache. 

# DEBUGGING COMMANDS

✅ View All Resources
```shell
kubectl get all
```
✅ Describe Pod
```shell
kubectl describe pod <pod-name>
```
✅ View Logs
```shell
kubectl logs <pod-name>
```


## Step 26: Process of deletion:
```shell
# Delete ingress controller
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.3/deploy/static/provider/aws/deploy.yaml
kubectl delete -f .  #Delete Application
```
 **Delete Cluster**
```shell
kops delete cluster \
--name=kopsvprofile.ehmutyam.xyz \
--state=s3://kopsstatebkt7526 \
--yes
```

**Delete manually:**
	Delete the hosted zone in route 53
	Delete the created records 4 NS and 1 CNAME record in domain registrar.
	Delete the s3 bucket created to store the kops state.
	Terminate the Kops instance if not required.
