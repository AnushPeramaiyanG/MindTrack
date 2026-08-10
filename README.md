# 🧠 MindTrack – Brain Tasks Application

A complete DevOps and Cloud deployment project demonstrating the containerization, CI/CD automation, Kubernetes deployment, AWS ECR image management, Amazon EKS orchestration, Application Load Balancer (ALB) integration, and monitoring using AWS CloudWatch.

---

## 📌 Project Overview

**MindTrack – Brain Tasks Application** is a web-based application deployed on AWS using a modern DevOps workflow.

The project implements:

* GitHub for source-code management
* Docker for application containerization
* Amazon ECR for Docker image storage
* Amazon EKS for Kubernetes orchestration
* AWS Load Balancer Controller for ALB provisioning
* Kubernetes Ingress for external application access
* AWS CodeBuild for automated build and deployment
* AWS CodePipeline for CI/CD automation
* AWS CloudWatch for monitoring and logs

### Deployment Architecture

```text
Developer
   │
   ▼
GitHub Repository
   │
   │ Webhook / Pipeline Trigger
   ▼
AWS CodePipeline
   │
   ▼
AWS CodeBuild
   │
   ├── Docker Build
   │
   ├── ECR Login
   │
   └── Docker Image Push
   │
   ▼
Amazon ECR
   │
   │ Docker Image
   ▼
Amazon EKS
   │
   ├── Deployment
   │       │
   │       └── Brain Task Pods
   │
   ├── ClusterIP Service
   │
   └── Ingress
   │
   ▼
AWS Load Balancer Controller
   │
   ▼
Application Load Balancer (ALB)
   │
   ▼
Internet
   │
   ▼
MindTrack Application
```

---

# 🛠️ Technologies Used

| Category            | Technology                    |
| ------------------- | ----------------------------- |
| Source Code         | GitHub                        |
| Version Control     | Git                           |
| Application         | Brain Tasks App               |
| Containerization    | Docker                        |
| Container Registry  | Amazon ECR                    |
| Orchestration       | Kubernetes                    |
| Kubernetes Platform | Amazon EKS                    |
| Load Balancer       | AWS Application Load Balancer |
| Controller          | AWS Load Balancer Controller  |
| CI/CD Build         | AWS CodeBuild                 |
| CI/CD Pipeline      | AWS CodePipeline              |
| Monitoring          | AWS CloudWatch                |
| Infrastructure      | AWS                           |
| Operating System    | Ubuntu / Amazon Linux         |
| Region              | `ap-south-1`                  |

---

# 📁 Project Structure

```text
MindTrack/
│
└── Brain-Tasks-App/
    │
    ├── Dockerfile
    ├── buildspec.yml
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── README.md
    │
    └── application files
```

---

# ☁️ AWS Environment

### AWS Region

```text
ap-south-1
```

### EKS Cluster

```text
brain-task-cluster
```

### AWS Account

The AWS account ID should not be hard-coded in public documentation or source files.

Use the following format when documenting resources:

```text
AWS Account: <AWS-ACCOUNT-ID>
```

---

# 1. 🔧 GitHub Repository

The project source code is maintained in GitHub.

Basic Git workflow:

```bash
git clone <GITHUB-REPOSITORY-URL>

cd MindTrack/Brain-Tasks-App
```

Check repository status:

```bash
git status
```

Add files:

```bash
git add .
```

Commit:

```bash
git commit -m "Update MindTrack DevOps deployment"
```

Push:

```bash
git push origin main
```

---

# 2. 🐳 Docker Containerization

The application is containerized using Docker.

## Dockerfile

Example:

```dockerfile
FROM nginx:alpine

COPY dist/ /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

The Docker image packages the application and Nginx web server into a portable container.

---

## Build Docker Image

```bash
docker build -t brain-task-app:latest .
```

Verify:

```bash
docker images
```

---

## Run Container Locally

```bash
docker run -d \
  --name brain-task-app \
  -p 8080:80 \
  brain-task-app:latest
```

Check:

```bash
docker ps
```

Test:

```text
http://localhost:8080
```

---

# 3. 📦 Amazon ECR

Amazon Elastic Container Registry (ECR) is used to store the Docker image.

## Create ECR Repository

```bash
aws ecr create-repository \
  --repository-name brain-task-app \
  --region ap-south-1
```

Verify:

```bash
aws ecr describe-repositories \
  --repository-names brain-task-app \
  --region ap-south-1
```

---

## Authenticate Docker with ECR

```bash
aws ecr get-login-password \
  --region ap-south-1 | \
docker login \
  --username AWS \
  --password-stdin \
  <AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com
```

---

## Tag Docker Image

```bash
docker tag brain-task-app:latest \
<AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
```

---

## Push Image to ECR

```bash
docker push \
<AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
```

Verify:

```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

---

# 4. ☸️ Amazon EKS

The application is deployed to Amazon EKS.

## Verify AWS CLI

```bash
aws --version
```

## Verify EKS Cluster

```bash
aws eks describe-cluster \
  --name brain-task-cluster \
  --region ap-south-1
```

---

## Configure kubectl

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name brain-task-cluster
```

Verify:

```bash
kubectl get nodes
```

Expected:

```text
NAME                                      STATUS   ROLES
ip-xxx.ap-south-1.compute.internal        Ready    <none>
ip-xxx.ap-south-1.compute.internal        Ready    <none>
```

---

# 5. 🚀 Kubernetes Deployment

The application is deployed using a Kubernetes Deployment.

## deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: brain-task-app
spec:
  replicas: 2

  selector:
    matchLabels:
      app: brain-task-app

  template:
    metadata:
      labels:
        app: brain-task-app

    spec:
      containers:
        - name: brain-task-app
          image: <AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest

          ports:
            - containerPort: 80
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployments
```

Check pods:

```bash
kubectl get pods
```

Expected:

```text
brain-task-app-xxxxxxxxxx   1/1   Running
brain-task-app-yyyyyyyyyy   1/1   Running
```

---

# 6. 🔗 Kubernetes Service

Because the project uses an AWS Application Load Balancer, the Kubernetes Service is configured as `ClusterIP`.

## service.yaml

```yaml
apiVersion: v1
kind: Service

metadata:
  name: brain-task-service

spec:
  selector:
    app: brain-task-app

  type: ClusterIP

  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f service.yaml
```

Verify:

```bash
kubectl get svc
```

Expected:

```text
NAME                 TYPE        CLUSTER-IP
brain-task-service   ClusterIP   10.x.x.x
```

---

# 7. ⚖️ AWS Application Load Balancer

Instead of using a Kubernetes `LoadBalancer` Service, the project uses an AWS Application Load Balancer through Kubernetes Ingress.

Architecture:

```text
Internet
   │
   ▼
AWS Application Load Balancer
   │
   ▼
Kubernetes Ingress
   │
   ▼
ClusterIP Service
   │
   ▼
Brain Task Pods
```

---

# 8. AWS Load Balancer Controller

The AWS Load Balancer Controller is installed in the EKS cluster.

Verify:

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

Expected:

```text
NAME                           READY
aws-load-balancer-controller   2/2
```

Check pods:

```bash
kubectl get pods -n kube-system | grep aws-load-balancer
```

Expected:

```text
aws-load-balancer-controller-xxxxx   1/1   Running
aws-load-balancer-controller-yyyyy   1/1   Running
```

The controller watches Kubernetes Ingress resources and provisions the AWS Application Load Balancer.

---

# 9. 🌐 Kubernetes Ingress

## ingress.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: brain-task-ingress

  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip

spec:
  ingressClassName: alb

  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: brain-task-service
                port:
                  number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

Check:

```bash
kubectl get ingress
```

Expected:

```text
NAME                 CLASS   HOSTS   ADDRESS
brain-task-ingress   alb     *       k8s-default-brain-task-xxxxx.ap-south-1.elb.amazonaws.com
```

---

# 10. 🔍 Verify Application Endpoints

Check Service endpoints:

```bash
kubectl get endpoints brain-task-service
```

Expected:

```text
NAME                 ENDPOINTS
brain-task-service   192.168.x.x:80,192.168.x.x:80
```

If endpoints are empty, check:

```bash
kubectl get pods --show-labels
```

The Pods must have:

```text
app=brain-task-app
```

because the Service selector is:

```yaml
selector:
  app: brain-task-app
```

---

# 11. 🔗 Get ALB DNS Name

Run:

```bash
kubectl get ingress brain-task-ingress
```

Or:

```bash
kubectl get ingress brain-task-ingress \
  -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

Example:

```text
k8s-default-brain-task-xxxxx.ap-south-1.elb.amazonaws.com
```

The application can then be accessed using:

```text
http://<ALB-DNS-NAME>
```

---

# 12. 🆔 Get Application Load Balancer ARN

The ALB ARN is retrieved using AWS CLI.

```bash
aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --query 'LoadBalancers[*].[LoadBalancerName,DNSName,LoadBalancerArn]' \
  --output table
```

To retrieve only the ARN:

```bash
aws elbv2 describe-load-balancers \
  --region ap-south-1 \
  --query 'LoadBalancers[*].LoadBalancerArn' \
  --output text
```

Example:

```text
arn:aws:elasticloadbalancing:ap-south-1:<AWS-ACCOUNT-ID>:loadbalancer/app/k8s-default-brain-task-xxxxx/xxxxxxxx
```

### Submission

```text
Application Load Balancer ARN:
<PASTE-ALB-ARN-HERE>
```

---

# 13. 🔄 AWS CodeBuild

AWS CodeBuild is used to automate:

1. ECR authentication
2. Docker image build
3. Docker image tagging
4. Docker image push
5. EKS authentication
6. Kubernetes deployment

This addresses the complete CI/CD build and deployment requirement.

---

# 14. 📄 buildspec.yml

Example:

```yaml
version: 0.2

phases:

  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin <AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com

      - REPOSITORY_URI=<AWS-ACCOUNT-ID>.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app
      - IMAGE_TAG=latest

  build:
    commands:
      - echo Building Docker image...
      - docker build -t $REPOSITORY_URI:$IMAGE_TAG .

      - echo Pushing Docker image to ECR...
      - docker push $REPOSITORY_URI:$IMAGE_TAG

  post_build:
    commands:
      - echo Updating kubeconfig...
      - aws eks update-kubeconfig --region ap-south-1 --name brain-task-cluster

      - echo Deploying Kubernetes application...
      - kubectl apply -f deployment.yaml
      - kubectl apply -f service.yaml
      - kubectl apply -f ingress.yaml

      - echo Checking deployment...
      - kubectl rollout status deployment/brain-task-app

      - echo Kubernetes resources:
      - kubectl get pods
      - kubectl get svc
      - kubectl get ingress
```

---

# 15. 🔄 CI/CD Pipeline

The complete pipeline is:

```text
GitHub
   │
   ▼
CodePipeline
   │
   ▼
CodeBuild
   │
   ├── Docker Build
   │
   ├── ECR Login
   │
   ├── Docker Push
   │
   ├── EKS Authentication
   │
   └── kubectl apply
          │
          ├── deployment.yaml
          ├── service.yaml
          └── ingress.yaml
                 │
                 ▼
               EKS
                 │
                 ▼
                ALB
```

Whenever application changes are pushed to GitHub, the pipeline can automatically trigger the build and deployment process.

---

# 16. 🔐 IAM Permissions

The project uses IAM roles/policies to provide required AWS access.

Required services include:

* Amazon ECR
* Amazon EKS
* IAM
* Amazon EC2
* VPC
* AWS CodeBuild
* AWS CodePipeline
* Amazon CloudWatch
* Elastic Load Balancing

The CodeBuild service role must have permissions required to:

* Authenticate to ECR
* Push Docker images
* Access EKS
* Update Kubernetes resources

The EKS worker-node role requires permissions to pull private images from ECR.

The AWS Load Balancer Controller uses an IAM role associated with its Kubernetes service account.

---

# 17. 📊 CloudWatch Monitoring

AWS CloudWatch is used for monitoring application and infrastructure logs.

Monitoring includes:

* CodeBuild logs
* CodePipeline execution
* Application logs
* EKS-related logs
* Deployment status
* Application health

Check CodeBuild logs from:

```text
AWS Console
→ CodeBuild
→ Build projects
→ Brain-Tasks-CodeBuild
→ Build history
→ Build
→ Logs
```

CloudWatch Logs can also be used to investigate deployment failures and application issues.

---

# 18. 🩺 Kubernetes Health Checks

Check all resources:

```bash
kubectl get all
```

Check pods:

```bash
kubectl get pods
```

Check deployments:

```bash
kubectl get deployments
```

Check services:

```bash
kubectl get svc
```

Check ingress:

```bash
kubectl get ingress
```

Check events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Describe failed pods:

```bash
kubectl describe pod <POD-NAME>
```

Check application logs:

```bash
kubectl logs <POD-NAME>
```

---

# 19. 🐳 Docker Image Verification

Check local images:

```bash
docker images
```

Check ECR images:

```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

Check detailed image information:

```bash
aws ecr describe-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

---

# 20. 🔧 Troubleshooting

## ImagePullBackOff

Check:

```bash
kubectl describe pod <POD-NAME>
```

Verify the ECR image exists:

```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

Verify the image tag in `deployment.yaml` matches the tag in ECR.

---

## Ingress Address Empty

Check:

```bash
kubectl describe ingress brain-task-ingress
```

Check the controller:

```bash
kubectl get pods -n kube-system | grep aws-load-balancer
```

Check controller logs:

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

---

## Pods Not Running

```bash
kubectl get pods
```

Then:

```bash
kubectl describe pod <POD-NAME>
```

Check the Events section.

---

## Service Has No Endpoints

```bash
kubectl get endpoints brain-task-service
```

Check Pod labels:

```bash
kubectl get pods --show-labels
```

The label must match:

```yaml
app: brain-task-app
```

---

# 21. 🧹 Cleanup

Delete Kubernetes resources:

```bash
kubectl delete -f ingress.yaml
kubectl delete -f service.yaml
kubectl delete -f deployment.yaml
```

Delete ECR repository if required:

```bash
aws ecr delete-repository \
  --repository-name brain-task-app \
  --region ap-south-1 \
  --force
```

Application should satisfy:

```text
EKS Nodes                         ✅ Ready
Pods                              ✅ Running
Deployment                        ✅ Available
Service                           ✅ ClusterIP
AWS LB Controller                 ✅ Running
Ingress                           ✅ Created
ALB                               ✅ Created
ALB DNS                           ✅ Available
ALB ARN                           ✅ Available
ECR Image                         ✅ Available
CodeBuild                         ✅ Successful
CodePipeline                      ✅ Successful
CloudWatch                        ✅ Monitoring
Application                       ✅ Accessible
```

---

# 24. 🎯 Project Outcome

The MindTrack Brain Tasks application demonstrates an end-to-end DevOps implementation on AWS.

The project successfully integrates:

```text
GitHub
   ↓
AWS CodePipeline
   ↓
AWS CodeBuild
   ↓
Docker
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes Deployment
   ↓
ClusterIP Service
   ↓
AWS Load Balancer Controller
   ↓
Application Load Balancer
   ↓
Internet
```

This implementation provides an automated and repeatable CI/CD workflow for deploying the Brain Tasks application to a Kubernetes cluster with AWS-native services and monitoring.

The complete Project Steps and Relevant Screenshots are documented and attached to this repository.
