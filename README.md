# Tech Challenge 2 - AWS EKS DevOps & GitOps Deployment

A hands-on DevOps and GitOps project deploying a containerized Python Flask application to Amazon EKS.

The project uses Terraform for AWS infrastructure, Docker for containerization, Amazon ECR for image storage, Helm for Kubernetes packaging, GitHub Actions for continuous integration, and Argo CD for GitOps-based continuous deployment.

The final application is exposed publicly through an AWS Application Load Balancer and automatically deployed whenever application changes are pushed to GitHub.

---

## Project Overview

This project demonstrates a complete cloud-native deployment workflow.

The application starts as Python source code and moves through an automated pipeline:

```text
Developer
    |
    v
GitHub
    |
    v
GitHub Actions
    |
    +---- Build Docker Image
    |
    +---- Push Image to Amazon ECR
    |
    +---- Update Helm Image Tag
              |
              v
            GitHub
              |
              v
           Argo CD
              |
              v
             Helm
              |
              v
          Amazon EKS
              |
              v
        Kubernetes Pod
              |
              v
           Service
              |
              v
           Ingress
              |
              v
AWS Load Balancer Controller
              |
              v
Application Load Balancer
              |
              v
       Public Application
```

---

## Tech Stack

| Technology | Purpose |
|---|---|
| Python | Application language |
| Flask | Web application framework |
| Docker | Containerizes the Flask application |
| Amazon ECR | Stores Docker images |
| Terraform | Provisions AWS infrastructure |
| Amazon VPC | Provides cloud networking |
| Amazon EKS | Managed Kubernetes cluster |
| Kubernetes | Runs and manages application containers |
| Helm | Packages Kubernetes manifests |
| Horizontal Pod Autoscaler | Automatically scales application pods |
| Metrics Server | Supplies CPU and memory metrics to the HPA |
| AWS Load Balancer Controller | Creates and manages the AWS ALB |
| Application Load Balancer | Exposes the application publicly |
| GitHub | Source control and GitOps source of truth |
| GitHub Actions | Continuous integration |
| GitHub OIDC | Provides secure GitHub-to-AWS authentication |
| Argo CD | GitOps continuous deployment |
| Jenkins | CI/CD pipeline implementation completed during the original project |

---

# 1. Flask Application

The project uses a simple Python Flask application.

Example:

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, World!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

The Flask application listens on port:

```text
5000
```

During final GitOps validation, the application was changed to return:

```text
Hello, World! GitOps deployment successful!
```

This allowed the automated deployment workflow to be verified from GitHub all the way to the public AWS load balancer.

---

# 2. Docker Containerization

The Flask application is packaged into a Docker image.

The Dockerfile installs the application's Python dependencies, copies the application source code into the image, exposes port `5000`, and starts the Flask application.

Basic Docker workflow:

```text
Application Code
      |
      v
Dockerfile
      |
      v
Docker Image
      |
      v
Container
```

The image can be built locally using:

```bash
docker build -t tech-challenge-2 .
```

The container can be started using:

```bash
docker run -d -p 5000:5000 tech-challenge-2
```

---

# 3. Amazon ECR

Amazon Elastic Container Registry is used as the private Docker registry for the project.

Repository:

```text
tech-challenge-2
```

Repository URI:

```text
099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2
```

Instead of relying only on the `latest` tag, the GitHub Actions pipeline tags images using the GitHub Actions run number.

Example:

```text
tech-challenge-2:3
tech-challenge-2:4
```

This gives each deployment a unique image version.

---

# 4. Terraform Infrastructure

Terraform is used to provision the AWS infrastructure required by the project.

The Terraform configuration includes resources for:

- VPC networking
- Public subnets
- Internet connectivity
- Security groups
- Amazon ECR
- Amazon EKS
- EKS managed node group
- Jenkins EC2 infrastructure
- IAM configuration

Basic Terraform workflow:

```text
Terraform Code
     |
     v
terraform init
     |
     v
terraform validate
     |
     v
terraform plan
     |
     v
terraform apply
     |
     v
AWS Infrastructure
```

Commands:

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

Terraform allows the infrastructure to be recreated consistently instead of manually rebuilding AWS resources through the console.

---

# 5. Amazon EKS

Amazon Elastic Kubernetes Service provides the Kubernetes environment for the application.

Cluster:

```text
tech-challenge-2-cluster
```

Node group:

```text
tech-challenge-2-nodes
```

The cluster uses managed EC2 worker nodes.

During the final deployment, the cluster was scaled to two worker nodes to provide enough pod capacity for the application, Argo CD, Metrics Server, and other Kubernetes components.

Cluster verification:

```bash
kubectl get nodes
```

Example healthy state:

```text
NAME                                       STATUS
ip-10-0-1-188.us-east-2.compute.internal   Ready
ip-10-0-2-163.us-east-2.compute.internal   Ready
```

---

# 6. Helm Deployment

Helm is used to package and manage the Kubernetes application configuration.

Chart location:

```text
helm/tech-challenge-2
```

The Helm chart manages resources including:

- Deployment
- Service
- ServiceAccount
- Ingress
- HorizontalPodAutoscaler

The container image configuration is stored inside:

```text
helm/tech-challenge-2/values.yaml
```

Example:

```yaml
image:
  repository: 099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2
  pullPolicy: IfNotPresent
  tag: "4"
```

The image tag is automatically updated by GitHub Actions.

---

# 7. Kubernetes Deployment

The Kubernetes Deployment manages the Flask application pods.

The final deployment used the versioned ECR image:

```text
099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:4
```

Deployment verification:

```bash
kubectl get deployment tech-challenge-2
```

Pod verification:

```bash
kubectl get pods
```

Example:

```text
NAME                                READY   STATUS
tech-challenge-2-86447f8657-qhhxd   1/1     Running
```

---

# 8. Kubernetes Service

The application uses a Kubernetes Service to route traffic to the Flask pod.

The Service exposes port:

```text
80
```

and forwards traffic to the Flask application running on:

```text
5000
```

Traffic flow:

```text
Service :80
     |
     v
Pod :5000
     |
     v
Flask Application
```

---

# 9. Horizontal Pod Autoscaler

The project uses a Kubernetes Horizontal Pod Autoscaler.

Configuration:

```text
Minimum Pods: 1
Maximum Pods: 3
CPU Target: 50%
Memory Target: 50%
```

Verification:

```bash
kubectl get hpa
```

Example:

```text
NAME               TARGETS                        MINPODS   MAXPODS   REPLICAS
tech-challenge-2   cpu: 1%/50%, memory: 19%/50%   1         3         1
```

The HPA allows Kubernetes to automatically increase or decrease application replicas based on resource utilization.

---

# 10. Metrics Server

The Horizontal Pod Autoscaler requires resource metrics.

Metrics Server was installed into the cluster to provide CPU and memory utilization data.

Installation:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Verification:

```bash
kubectl get pods -n kube-system
```

Example:

```text
metrics-server-687f9dc499-bvqnl   1/1   Running
```

Metrics can be viewed using:

```bash
kubectl top pods
```

Example:

```text
NAME                               CPU(cores)   MEMORY(bytes)
tech-challenge-2-c5f7ff9fd-jsf9c   1m           24Mi
```

The relationship is:

```text
Application Pods
      |
      v
Metrics Server
      |
      v
CPU / Memory Metrics
      |
      v
HPA
      |
      v
Scale Pods When Needed
```

---

# 11. AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS load balancers from Kubernetes resources.

The Kubernetes Ingress uses:

```text
Ingress Class: alb
```

The Ingress includes:

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
alb.ingress.kubernetes.io/target-type: ip
```

The controller watches the Kubernetes Ingress and provisions an AWS Application Load Balancer.

Architecture:

```text
Kubernetes Ingress
       |
       v
AWS Load Balancer Controller
       |
       v
AWS Application Load Balancer
```

The controller runs inside the `kube-system` namespace.

Verification:

```bash
kubectl get pods -n kube-system
```

Example:

```text
aws-load-balancer-controller-5b888ff8c9-2j4p2   1/1   Running
aws-load-balancer-controller-5b888ff8c9-xgscj   1/1   Running
```

---

# 12. EKS OIDC and IAM

The AWS Load Balancer Controller requires permission to interact with AWS services.

The project uses IAM Roles for Service Accounts with an EKS OIDC identity provider.

After rebuilding the EKS cluster, a new OIDC provider was required because the new cluster had a different OIDC issuer.

The IAM role:

```text
AmazonEKSLoadBalancerControllerRole
```

was updated to trust the new EKS OIDC provider.

The Kubernetes service account:

```text
kube-system/aws-load-balancer-controller
```

was annotated with the IAM role:

```text
arn:aws:iam::099771438874:role/AmazonEKSLoadBalancerControllerRole
```

This allows the AWS Load Balancer Controller pod to securely receive AWS permissions without storing AWS access keys inside Kubernetes.

---

# 13. Application Load Balancer

Once the AWS Load Balancer Controller became healthy, the Kubernetes Ingress received a public AWS ALB address.

Ingress verification:

```bash
kubectl get ingress
```

Example:

```text
NAME               CLASS   HOSTS   ADDRESS
tech-challenge-2   alb     *       k8s-default-techchal-b6324cb6b9-938526015.us-east-2.elb.amazonaws.com
```

Traffic flow:

```text
Internet
   |
   v
AWS Application Load Balancer
   |
   v
Kubernetes Ingress
   |
   v
Kubernetes Service
   |
   v
Flask Pod :5000
```

---

# 14. GitHub Actions Continuous Integration

GitHub Actions provides the CI portion of the GitOps workflow.

Workflow:

```text
.github/workflows/ci.yml
```

The workflow runs when code is pushed to:

```text
main
```

The pipeline performs the following operations:

1. Checks out the repository
2. Authenticates to AWS
3. Logs in to Amazon ECR
4. Builds the Docker image
5. Tags the image using the GitHub Actions run number
6. Pushes the image to Amazon ECR
7. Updates the Helm image tag
8. Commits the updated `values.yaml` back to GitHub

Pipeline flow:

```text
git push
   |
   v
GitHub Actions
   |
   v
Build Docker Image
   |
   v
Tag Image
   |
   v
Amazon ECR
   |
   v
Update values.yaml
   |
   v
Commit New Desired State
```

---

# 15. GitHub OIDC Authentication

GitHub Actions authenticates to AWS using OpenID Connect instead of storing permanent AWS access keys inside GitHub.

GitHub Actions assumes:

```text
arn:aws:iam::099771438874:role/GitHubActionsTechChallenge2Role
```

The workflow requires:

```yaml
permissions:
  contents: write
  id-token: write
```

AWS authentication:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::099771438874:role/GitHubActionsTechChallenge2Role
    aws-region: us-east-2
```

Authentication flow:

```text
GitHub Actions
      |
      v
GitHub OIDC Token
      |
      v
AWS IAM Trust Policy
      |
      v
GitHubActionsTechChallenge2Role
      |
      v
Temporary AWS Credentials
      |
      v
Amazon ECR
```

This avoids storing long-lived AWS access keys in the repository or workflow.

---

# 16. Argo CD GitOps Continuous Deployment

Argo CD provides the continuous deployment portion of the project.

Argo CD runs inside the EKS cluster in the:

```text
argocd
```

namespace.

Installation:

```bash
kubectl create namespace argocd
```

Argo CD was installed using server-side apply:

```bash
kubectl apply -n argocd \
  --server-side \
  --force-conflicts \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Server-side apply was used because the ApplicationSet CRD exceeded the annotation size supported by client-side apply.

Argo CD components include:

```text
argocd-application-controller
argocd-applicationset-controller
argocd-dex-server
argocd-notifications-controller
argocd-redis
argocd-repo-server
argocd-server
```

---

# 17. Argo CD Application

An Argo CD Application resource connects the GitHub repository to the Kubernetes cluster.

Application:

```text
tech-challenge-2
```

Repository:

```text
https://github.com/folsedo/tech-challenge-2.git
```

Branch:

```text
main
```

Helm chart:

```text
helm/tech-challenge-2
```

Example configuration:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: tech-challenge-2
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/folsedo/tech-challenge-2.git
    targetRevision: main
    path: helm/tech-challenge-2
    helm:
      valueFiles:
        - values.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

The application is created using:

```bash
kubectl apply -f argocd-application.yaml
```

---

# 18. Automated Argo CD Synchronization

Argo CD continuously compares:

```text
Git Desired State
        vs
Kubernetes Actual State
```

When they differ, Argo CD automatically synchronizes the cluster.

The project uses:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

### Automated Sync

Argo CD automatically deploys Git changes.

### Self Heal

If Kubernetes configuration is manually changed and no longer matches Git, Argo CD restores the Git-defined configuration.

### Prune

If a Kubernetes resource is removed from Git, Argo CD can remove the corresponding resource from the cluster.

This makes Git the source of truth for the Kubernetes application.

---

# 19. Complete GitOps Workflow

The final CI/CD architecture is:

```text
Developer
   |
   | git push
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   +-------------------------------+
   |                               |
   v                               v
Build Docker Image          Authenticate to AWS
   |                           using OIDC
   |                               |
   +---------------+---------------+
                   |
                   v
              Amazon ECR
                   |
                   v
          Push Versioned Image
                   |
                   v
        Update Helm values.yaml
                   |
                   v
             Commit to Git
                   |
                   v
                Argo CD
                   |
            Detect Git Change
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
                   v
            Application Pod
                   |
                   v
               Service
                   |
                   v
               Ingress
                   |
                   v
      AWS Load Balancer Controller
                   |
                   v
       Application Load Balancer
                   |
                   v
               Internet
```

---

# 20. GitOps Validation Test

The final workflow was tested by changing the Flask application response.

Original:

```text
Hello, World!
```

Updated:

```text
Hello, World! GitOps deployment successful!
```

Only the following Git commands were used to deploy the change:

```bash
git add app.py
git commit -m "Test automated GitOps deployment"
git push origin main
```

No manual:

```text
kubectl apply
helm upgrade
docker push
```

was required for the application update.

GitHub Actions automatically:

```text
Built the new Docker image
        |
        v
Pushed the image to ECR
        |
        v
Updated values.yaml
        |
        v
Committed the image tag
```

Argo CD then automatically:

```text
Detected the Git change
        |
        v
Synchronized Helm
        |
        v
Updated the EKS Deployment
        |
        v
Created the new application Pod
```

The Kubernetes deployment automatically moved to image:

```text
099771438874.dkr.ecr.us-east-2.amazonaws.com/tech-challenge-2:4
```

---

# 21. Final Validation

Argo CD:

```bash
kubectl get applications -n argocd
```

Result:

```text
NAME               SYNC STATUS   HEALTH STATUS
tech-challenge-2   Synced        Healthy
```

Application pod:

```bash
kubectl get pods
```

Result:

```text
NAME                                READY   STATUS
tech-challenge-2-86447f8657-qhhxd   1/1     Running
```

HPA:

```bash
kubectl get hpa
```

Result:

```text
TARGETS
cpu: 1%/50%
memory: 19%/50%

MINPODS: 1
MAXPODS: 3
```

Worker nodes:

```bash
kubectl get nodes
```

Result:

```text
2 Kubernetes worker nodes Ready
```

Ingress:

```bash
kubectl get ingress
```

Result:

```text
CLASS: alb
PORT: 80
AWS ALB address successfully provisioned
```

Public application test:

```bash
curl http://<ALB-DNS-NAME>
```

Result:

```text
Hello, World! GitOps deployment successful!
```

---

# 22. Troubleshooting Performed

Several real-world issues were encountered and resolved during the project.

### EKS Pod Capacity

Problem:

```text
0/1 nodes are available: Too many pods
```

Solution:

The EKS managed node group was scaled from one desired worker node to two.

---

### ImagePullBackOff

Problem:

```text
ImagePullBackOff
```

Cause:

The ECR repository had been recreated and did not contain the image referenced by the Helm chart.

Solution:

GitHub Actions successfully built and pushed a new versioned Docker image to ECR.

---

### GitHub Actions AWS Authentication

Problem:

```text
Credentials could not be loaded
Could not load credentials from any providers
```

Cause:

The GitHub Actions AWS credentials configuration and OIDC trust configuration required correction.

Solution:

GitHub OIDC authentication was configured using:

```text
GitHub Actions
      |
      v
OIDC
      |
      v
AWS IAM Role
```

The workflow was corrected to use:

```yaml
role-to-assume: arn:aws:iam::099771438874:role/GitHubActionsTechChallenge2Role
aws-region: us-east-2
```

---

### Argo CD Degraded

Problem:

```text
Synced / Degraded
```

The application Deployment eventually became healthy, but other Kubernetes resources still required supporting controllers.

---

### HPA Metrics Unknown

Problem:

```text
cpu: <unknown>/50%
memory: <unknown>/50%
```

Cause:

Metrics Server was not installed after the EKS cluster rebuild.

Solution:

Metrics Server was installed.

Final result:

```text
cpu: 1%/50%
memory: 19%/50%
```

---

### ALB Ingress Without Address

Problem:

```text
Ingress Class: alb
Address:
```

Cause:

The AWS Load Balancer Controller was not installed on the rebuilt cluster.

The existing IAM role also trusted the OIDC provider belonging to the previous EKS cluster.

Solution:

1. Registered the new EKS OIDC provider
2. Updated the Load Balancer Controller IAM role trust relationship
3. Created and annotated the Kubernetes service account
4. Installed the AWS Load Balancer Controller using Helm
5. Specified the current cluster name, AWS region, and VPC ID

Final result:

```text
Ingress
   |
   v
AWS ALB successfully provisioned
```

---

# 23. Key DevOps Concepts Demonstrated

This project demonstrates:

- Infrastructure as Code
- Containerization
- Container registries
- Kubernetes orchestration
- Kubernetes deployments
- Kubernetes services
- Kubernetes ingress
- Horizontal pod autoscaling
- Kubernetes metrics
- AWS IAM
- IAM Roles for Service Accounts
- OpenID Connect
- CI pipelines
- GitOps
- Continuous deployment
- Automated reconciliation
- Application Load Balancing
- Cloud networking
- Troubleshooting across multiple infrastructure layers

---

# 24. CI vs CD in This Project

A simple way to understand the workflow:

### GitHub Actions = CI

GitHub Actions handles:

```text
Code
 ↓
Build
 ↓
Docker Image
 ↓
ECR
```

### Argo CD = CD

Argo CD handles:

```text
Git Desired State
 ↓
Helm
 ↓
Kubernetes
 ↓
Running Application
```

Together:

```text
GitHub Actions
      |
      v
     ECR
      |
      v
Git Configuration
      |
      v
    Argo CD
      |
      v
     EKS
```

---

# 25. Project Outcome

The final environment successfully demonstrated an automated cloud-native DevOps and GitOps workflow.

A developer can change the application and run:

```bash
git add .
git commit -m "Update application"
git push origin main
```

The remaining deployment process happens automatically:

```text
GitHub Actions
     ↓
Docker Build
     ↓
Amazon ECR
     ↓
Helm Image Update
     ↓
Git Commit
     ↓
Argo CD
     ↓
Amazon EKS
     ↓
Application Load Balancer
     ↓
Updated Public Application
```

Final status:

```text
Terraform Infrastructure        PASS
Amazon EKS                     PASS
Docker Containerization        PASS
Amazon ECR                     PASS
Helm Deployment                PASS
Kubernetes Deployment          PASS
Horizontal Pod Autoscaler      PASS
Metrics Server                 PASS
AWS Load Balancer Controller   PASS
Application Load Balancer      PASS
GitHub Actions CI              PASS
GitHub OIDC Authentication     PASS
Argo CD GitOps CD              PASS
Automated Application Update   PASS
Public Application             PASS
```

---

## Final Result

```text
Hello, World! GitOps deployment successful!
```

The project successfully demonstrates an end-to-end AWS DevOps and GitOps deployment using Terraform, Docker, ECR, EKS, Kubernetes, Helm, GitHub Actions, Argo CD, HPA, Metrics Server, and an AWS Application Load Balancer.
