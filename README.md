\# Tech Challenge 2 — AWS EKS CI/CD Deployment



A hands-on DevOps project that deploys a containerized Python Flask application to Amazon EKS using Terraform, Docker, Amazon ECR, Helm, Kubernetes, AWS Application Load Balancer, Horizontal Pod Autoscaling, and Jenkins CI/CD.



\---



\## Project Overview



This project demonstrates an end-to-end cloud-native deployment workflow.



The application is a simple Python Flask application that returns:



```text

Hello, World!

```



The infrastructure and deployment process were automated using DevOps tools and AWS services.



\---



\## Architecture



```text

Developer

&#x20;   |

&#x20;   v

GitHub

&#x20;   |

&#x20;   v

Jenkins

&#x20;   |

&#x20;   +----> Docker Build

&#x20;   |

&#x20;   v

Amazon ECR

&#x20;   |

&#x20;   v

Helm

&#x20;   |

&#x20;   v

Amazon EKS

&#x20;   |

&#x20;   v

Kubernetes Deployment

&#x20;   |

&#x20;   +----> Horizontal Pod Autoscaler

&#x20;   |

&#x20;   v

AWS Application Load Balancer

&#x20;   |

&#x20;   v

Flask Application

```



\---



\## Technologies Used



| Technology | Purpose |

|---|---|

| Python | Application language |

| Flask | Web application framework |

| Docker | Containerization |

| Amazon ECR | Private Docker image registry |

| Terraform | AWS infrastructure provisioning |

| Amazon EKS | Managed Kubernetes cluster |

| Kubernetes | Container orchestration |

| Helm | Kubernetes package and deployment management |

| AWS Load Balancer Controller | Kubernetes integration with AWS ALB |

| Application Load Balancer | Public application access |

| Metrics Server | Kubernetes resource metrics |

| HPA | Automatic pod scaling |

| Jenkins | CI/CD automation |

| GitHub | Source control |



\---



\## Infrastructure



Terraform provisions the AWS infrastructure required for the project, including:



\- VPC

\- Public subnets

\- Internet Gateway

\- Route tables

\- Security groups

\- Amazon ECR repository

\- Amazon EKS cluster

\- EKS managed node group

\- IAM roles and policies

\- Jenkins EC2 instance

\- Jenkins IAM instance profile

\- Jenkins EKS access



The EKS managed node group uses:



```text

Instance Type: t3.small

Desired Nodes: 1

Minimum Nodes: 1

Maximum Nodes: 4

```



\---



\## Docker and Amazon ECR



The Flask application is packaged into a Docker image.



Jenkins automatically builds the image and pushes it to the private Amazon ECR repository.



Example image:



```text

099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:3

```



Unique Jenkins build numbers are used as Docker image tags so deployments can be traced back to the pipeline build that created them.



\---



\## Kubernetes and Helm



The application is deployed to Amazon EKS using Helm.



The Helm chart manages resources including:



\- Deployment

\- Service

\- Ingress

\- Horizontal Pod Autoscaler

\- Service Account



The application container listens on port `5000`, while the Kubernetes Service exposes the application internally through port `80`.



\---



\## Application Load Balancer



The AWS Load Balancer Controller was installed in the EKS cluster.



A Kubernetes Ingress resource provisions an AWS Application Load Balancer that exposes the Flask application publicly over HTTP port 80.



Deployment testing confirmed that the ALB successfully returned:



```text

Hello, World!

```



\---



\## Horizontal Pod Autoscaling



Kubernetes Metrics Server was installed to provide CPU and memory metrics.



The Horizontal Pod Autoscaler is configured with:



```text

Minimum Replicas: 1

Maximum Replicas: 3

CPU Target: 50%

Memory Target: 50%

```



Load testing confirmed that the application successfully scaled:



```text

1 Pod

&#x20; ↓

2 Pods

&#x20; ↓

3 Pods

```



After the load was removed, Kubernetes automatically scaled the deployment back down to one replica.



\---



\## Jenkins CI/CD Pipeline



Jenkins runs on an Amazon Linux 2023 EC2 instance.



The Jenkins server contains:



\- Git

\- Docker

\- AWS CLI

\- kubectl

\- Helm

\- Java 21

\- Jenkins



The EC2 instance uses an IAM role instead of storing permanent AWS access keys on the server.



The Jenkins IAM role provides access to Amazon ECR and Amazon EKS.



\---



\## CI/CD Workflow



The Jenkins pipeline performs the following workflow:



```text

Checkout SCM

&#x20;    |

&#x20;    v

Verify Tools

&#x20;    |

&#x20;    v

Build Docker Image

&#x20;    |

&#x20;    v

Login to Amazon ECR

&#x20;    |

&#x20;    v

Tag and Push Image

&#x20;    |

&#x20;    v

Connect to Amazon EKS

&#x20;    |

&#x20;    v

Deploy with Helm

&#x20;    |

&#x20;    v

Verify Kubernetes Deployment

```



The completed pipeline successfully:



1\. Pulled the project from GitHub

2\. Built the Docker image

3\. Authenticated with Amazon ECR

4\. Tagged the image using the Jenkins build number

5\. Pushed the image to ECR

6\. Connected to the EKS cluster

7\. Updated the application using Helm

8\. Verified the Kubernetes rollout



\---



\## Final CI/CD Verification



Jenkins Build #3 completed successfully.



The image generated by the pipeline was:



```text

tech-challenge-2:3

```



The EKS deployment was verified to be running:



```text

099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:3

```



This confirmed the complete CI/CD workflow:



```text

GitHub

&#x20;  |

&#x20;  v

Jenkins

&#x20;  |

&#x20;  v

Docker

&#x20;  |

&#x20;  v

Amazon ECR

&#x20;  |

&#x20;  v

Helm

&#x20;  |

&#x20;  v

Amazon EKS

&#x20;  |

&#x20;  v

AWS ALB

&#x20;  |

&#x20;  v

Flask Application

```



\---



\## Troubleshooting



Several real-world issues were identified and resolved during the project.



\### ECR Authentication



Docker Desktop initially returned authentication errors while logging into Amazon ECR.



The Docker ECR credential helper was configured to allow successful authentication and image pushes.



\### AWS Load Balancer Controller



The controller initially failed because it could not automatically determine the VPC ID.



The deployment was corrected by explicitly providing:



\- AWS region

\- EKS cluster name

\- VPC ID



\### Kubernetes Metrics



The Kubernetes Metrics API was initially unavailable.



Metrics Server was installed, allowing:



```bash

kubectl top nodes

kubectl top pods

```



and enabling HPA CPU and memory scaling.



\### Jenkins Java Version



Jenkins initially failed to start using Java 17.



Amazon Corretto Java 21 was installed and Jenkins started successfully.



\### Jenkins Disk Space



The Jenkins EC2 root volume was originally 8 GB.



Terraform was used to expand the EBS volume to 20 GB. The partition and XFS filesystem were then expanded from Linux.



\### Jenkins Docker Permissions



The Jenkins service account initially received permission errors when accessing the Docker daemon.



The Jenkins user was added to the Docker group, allowing the CI/CD pipeline to build Docker images.



\### EKS Authentication



The Jenkins IAM role could initially identify the EKS cluster but could not access Kubernetes resources.



The EKS authentication mode was updated to:



```text

API\_AND\_CONFIG\_MAP

```



An EKS access entry and cluster access policy were then configured for the Jenkins IAM role.



\---



\## Final Result



The completed project demonstrates:



\- Infrastructure as Code with Terraform

\- Docker containerization

\- Private image storage with Amazon ECR

\- Kubernetes orchestration with Amazon EKS

\- Helm-based application deployment

\- AWS Application Load Balancer integration

\- Kubernetes Horizontal Pod Autoscaling

\- IAM-based AWS authentication

\- Jenkins CI/CD automation

\- GitHub source control

\- End-to-end automated application deployment



The final Jenkins pipeline successfully built, pushed, deployed, and verified the application running in Amazon EKS.

