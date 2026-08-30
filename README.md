# Tech Challenge 2 — AWS EKS CI/CD Deployment

A hands-on DevOps project deploying a containerized Python Flask application to Amazon EKS using Terraform, Docker, Amazon ECR, Helm, Kubernetes, AWS Application Load Balancer, Horizontal Pod Autoscaling, and Jenkins CI/CD.

---

## What This Project Does

- Builds a Python Flask application
- Packages the application into a Docker image
- Stores Docker images in Amazon ECR
- Provisions AWS infrastructure using Terraform
- Creates an Amazon EKS Kubernetes cluster
- Deploys the application using Helm
- Exposes the application through an AWS Application Load Balancer
- Configures Kubernetes Horizontal Pod Autoscaling
- Runs Jenkins on Amazon EC2
- Automates build, push, and deployment through Jenkins CI/CD
- Uses IAM roles instead of permanent AWS credentials
- Demonstrates real-world troubleshooting across Docker, AWS, Kubernetes, Terraform, and Jenkins

---

## Tech Stack

| Tool | Purpose |
|---|---|
| Python | Application language |
| Flask | Web application framework |
| Docker | Containerization |
| Amazon ECR | Private container registry |
| Terraform | Infrastructure as Code |
| Amazon EKS | Managed Kubernetes cluster |
| Kubernetes | Container orchestration |
| Helm | Kubernetes package management |
| AWS ALB | Public application access |
| Metrics Server | Kubernetes resource metrics |
| HPA | Automatic pod scaling |
| Jenkins | CI/CD automation |
| Amazon EC2 | Jenkins server |
| IAM | AWS authentication and authorization |
| Git | Version control |
| GitHub | Source control hosting |

---

## Architecture

```text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
Docker Build
    |
    v
Amazon ECR
    |
    v
Helm
    |
    v
Amazon EKS
    |
    v
Kubernetes Deployment
    |
    +------> Horizontal Pod Autoscaler
    |
    v
AWS Application Load Balancer
    |
    v
Flask Application
```

---

## Repository Structure

```text
tech-challenge-2/
│
├── app/
│
├── helm/
│   └── tech-challenge-2/
│
├── terraform/
│
├── app.py
├── requirements.txt
├── Dockerfile
├── Jenkinsfile
├── .gitignore
└── README.md
```

---

## Infrastructure

Terraform provisions the AWS infrastructure required for the project.

```text
AWS VPC
│
├── Public Subnet A
├── Public Subnet B
├── Internet Gateway
├── Route Tables
├── Security Groups
│
├── Amazon EKS Cluster
│   └── Managed Node Group
│
├── Amazon ECR Repository
│
└── Jenkins EC2 Instance
```

### EKS Node Group Configuration

```text
Instance Type: t3.small
Desired Nodes: 1
Minimum Nodes: 1
Maximum Nodes: 4
```

---

## Docker and Amazon ECR

The Flask application is packaged into a Docker image.

Jenkins builds the image and pushes it to a private Amazon ECR repository.

Example deployment image:

```text
099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:3
```

Jenkins build numbers are used as Docker image tags so each deployment can be traced back to the pipeline build that created it.

---

## Kubernetes and Helm

The application is deployed to Amazon EKS using Helm.

The Helm chart manages:

- Deployment
- Service
- Ingress
- Horizontal Pod Autoscaler
- Service Account

The Flask container listens on:

```text
Port 5000
```

The Kubernetes Service exposes the application internally through:

```text
Port 80
```

---

## Application Load Balancer

The AWS Load Balancer Controller integrates Kubernetes with AWS Elastic Load Balancing.

A Kubernetes Ingress resource provisions an AWS Application Load Balancer.

```text
Internet
    |
    v
AWS ALB
    |
    v
Kubernetes Service
    |
    v
Flask Pod
```

Deployment testing confirmed that the public load balancer successfully returned:

```text
Hello, World!
```

---

## Horizontal Pod Autoscaling

Kubernetes Metrics Server provides CPU and memory metrics to the Horizontal Pod Autoscaler.

### HPA Configuration

```text
Minimum Replicas: 1
Maximum Replicas: 3
CPU Target: 50%
Memory Target: 50%
```

Load testing confirmed automatic scaling:

```text
1 Pod
  |
  v
2 Pods
  |
  v
3 Pods
```

After the load was removed, Kubernetes automatically scaled the application back to one pod.

---

## Jenkins CI/CD Pipeline

Jenkins runs on an Amazon Linux 2023 EC2 instance.

The Jenkins server contains:

- Git
- Docker
- AWS CLI
- kubectl
- Helm
- Java 21
- Jenkins

The EC2 instance uses an IAM role instead of permanent AWS access keys.

The IAM role provides Jenkins access to:

```text
Amazon ECR
Amazon EKS
```

---

## CI/CD Workflow

```text
GitHub
   |
   v
Jenkins
   |
   v
Build Docker Image
   |
   v
Login to Amazon ECR
   |
   v
Tag Image
   |
   v
Push Image to ECR
   |
   v
Connect to Amazon EKS
   |
   v
Deploy with Helm
   |
   v
Verify Kubernetes Deployment
```

The pipeline performs the following workflow:

1. Pulls the source code from GitHub
2. Verifies required tools
3. Builds the Docker image
4. Authenticates with Amazon ECR
5. Tags the image using the Jenkins build number
6. Pushes the image to ECR
7. Connects to the EKS cluster
8. Deploys the application using Helm
9. Verifies the Kubernetes rollout

---

## Final CI/CD Verification

Jenkins Build #3 completed successfully.

The pipeline created:

```text
tech-challenge-2:3
```

The running EKS deployment used:

```text
099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:3
```

This confirmed the full deployment workflow:

```text
GitHub
   |
   v
Jenkins
   |
   v
Docker
   |
   v
Amazon ECR
   |
   v
Helm
   |
   v
Amazon EKS
   |
   v
AWS ALB
   |
   v
Flask Application
```

---

## Troubleshooting

### ECR Authentication

**Problem**

Docker Desktop returned authentication errors while attempting to authenticate with Amazon ECR.

**Resolution**

Configured the Amazon ECR Docker credential helper.

**Result**

```text
Docker
   |
   v
Amazon ECR
   |
   v
Authenticated
```

---

### AWS Load Balancer Controller

**Problem**

The AWS Load Balancer Controller entered `CrashLoopBackOff`.

The controller could not automatically determine the VPC ID.

**Resolution**

The Helm configuration was updated with:

```text
AWS Region
EKS Cluster Name
VPC ID
```

**Result**

The controller successfully provisioned the AWS Application Load Balancer.

---

### Kubernetes Metrics Server

**Problem**

Kubernetes resource metrics were unavailable.

Commands such as:

```bash
kubectl top nodes
kubectl top pods
```

could not return metrics.

**Resolution**

Installed Kubernetes Metrics Server.

**Result**

CPU and memory metrics became available and the Horizontal Pod Autoscaler functioned correctly.

---

### Jenkins Java Version

**Problem**

Jenkins failed to start with the installed Java version.

**Resolution**

Installed Amazon Corretto Java 21.

**Result**

```text
Jenkins Service
     |
     v
   Running
```

---

### Jenkins Disk Space

**Problem**

The Jenkins EC2 root volume was originally:

```text
8 GB
```

**Resolution**

Terraform was used to increase the EBS volume to:

```text
20 GB
```

The Linux partition and XFS filesystem were then expanded.

---

### Jenkins Docker Permissions

**Problem**

The Jenkins service account could not access the Docker daemon.

**Resolution**

Added the Jenkins user to the Docker group.

**Result**

Jenkins successfully built Docker images from the pipeline.

---

### EKS Authentication

**Problem**

The Jenkins IAM role could identify the EKS cluster but could not access Kubernetes resources.

**Resolution**

The EKS authentication mode was updated to:

```text
API_AND_CONFIG_MAP
```

An EKS access entry and cluster access policy were then assigned to the Jenkins IAM role.

**Result**

Jenkins successfully deployed the application to Amazon EKS.

---

## Final Validation

- ✅ Flask Application Running
- ✅ Docker Image Built
- ✅ Amazon ECR Image Stored
- ✅ Terraform Infrastructure Provisioned
- ✅ Amazon EKS Cluster Running
- ✅ Managed Node Group Running
- ✅ Helm Deployment Successful
- ✅ AWS Application Load Balancer Working
- ✅ Metrics Server Running
- ✅ Horizontal Pod Autoscaler Working
- ✅ Jenkins CI/CD Pipeline Successful
- ✅ GitHub → Jenkins → ECR → Helm → EKS Deployment Verified

---

## Skills Demonstrated

- AWS
- Amazon EKS
- Amazon ECR
- Amazon EC2
- Terraform
- Docker
- Kubernetes
- Helm
- Jenkins
- CI/CD
- IAM
- AWS Load Balancer Controller
- Application Load Balancers
- Kubernetes Ingress
- Horizontal Pod Autoscaling
- Kubernetes Metrics
- Linux
- Git
- GitHub
- Infrastructure Troubleshooting

---

## Project Cleanup

After validation and documentation were completed, the AWS infrastructure was destroyed to prevent unnecessary cloud charges.

```bash
terraform destroy
```

Additional Kubernetes-created AWS resources were identified and removed during cleanup, including load balancer security groups.

Final Terraform result:

```text
Destroy complete!
```

---

## Lessons Learned

This project reinforced several important DevOps concepts:

- Terraform can provision and manage complete AWS environments.
- Docker packages applications consistently across environments.
- Amazon ECR provides private container image storage.
- Kubernetes manages application deployment and scaling.
- Helm simplifies Kubernetes application management.
- AWS Load Balancer Controller connects Kubernetes Ingress resources to AWS ALB.
- Horizontal Pod Autoscaling requires resource requests and metrics.
- IAM roles are safer than storing permanent AWS credentials.
- Jenkins can automate an entire application deployment workflow.
- Troubleshooting cloud infrastructure requires working through dependencies systematically.

---

## About This Project

This project demonstrates an end-to-end DevOps workflow combining Infrastructure as Code, containerization, Kubernetes orchestration, cloud networking, autoscaling, and CI/CD automation.

The goal was not simply to deploy an application, but to understand how the individual DevOps tools work together as a complete deployment system.

```text
Terraform → Infrastructure
Docker    → Application Package
ECR       → Image Registry
EKS       → Kubernetes Platform
Helm      → Application Deployment
ALB       → Public Traffic
HPA       → Automatic Scaling
Jenkins   → CI/CD Automation
```

---

## Author

**Farrell Shelton**

Cloud Engineer | AWS | Kubernetes | Linux | RHCSA | Terraform | Docker | Jenkins | Ansible

Building production-ready cloud infrastructure one project at a time.
