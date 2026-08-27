# 🚀 Trend — React App CI/CD Deployment on AWS EKS

<p align="center">
  <img src="https://img.shields.io/badge/React-18-blue?logo=react" alt="React" />
  <img src="https://img.shields.io/badge/Docker-Container-2496ED?logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/Terraform-IaC-844FBA?logo=terraform&logoColor=white" alt="Terraform" />
  <img src="https://img.shields.io/badge/Jenkins-CI%2FCD-D24939?logo=jenkins&logoColor=white" alt="Jenkins" />
  <img src="https://img.shields.io/badge/Kubernetes-AWS%20EKS-326CE5?logo=kubernetes&logoColor=white" alt="Kubernetes" />
  <img src="https://img.shields.io/badge/Monitoring-Prometheus%20%26%20Grafana-E6522C?logo=grafana&logoColor=white" alt="Monitoring" />
</p>

<p align="center">
  Taking a React application from source code to a fully automated, production-ready deployment — using Docker, Terraform, Jenkins, AWS EKS, and open-source monitoring.
</p>

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Tech Stack](#-tech-stack)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Setup Instructions](#-setup-instructions)
- [Pipeline Explanation](#-pipeline-explanation)
- [Live Deployment](#-live-deployment)
- [Screenshot Evidence](#-screenshot-evidence)
- [License](#-license)

---

## 🧭 Overview

This project takes a React application from source code to a fully automated production deployment:

```
GitHub push → Jenkins (build & test) → Docker image → DockerHub → Kubernetes (AWS EKS) → LoadBalancer → Live application
                                                                          ↑
                                                          Prometheus + Grafana (monitoring)
```

Every push to `main` automatically triggers Jenkins to build a new Docker image, push it to DockerHub, and roll it out to the Kubernetes cluster — **no manual steps required after the initial setup.**

---

## 🛠 Tech Stack

| Layer                   | Tool                        |
|-------------------------|------------------------------|
| Application             | React                        |
| Containerization        | Docker                       |
| Infrastructure as Code  | Terraform                    |
| CI/CD                   | Jenkins                      |
| Image Registry          | DockerHub                    |
| Orchestration           | Kubernetes on AWS EKS        |
| Monitoring              | Prometheus + Grafana         |

---

## 📁 Project Structure

```
Trend/
├── Dockerfile              # Multi-stage build for the React app
├── .dockerignore
├── .gitignore
├── Jenkinsfile             # Declarative CI/CD pipeline
├── terraform/
│   └── main.tf             # VPC, IAM, EC2 + Jenkins provisioning
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment (2 replicas)
│   └── service.yaml        # Kubernetes LoadBalancer Service
├── docs/
│   └── screenshots/        # Screenshots referenced below
└── README.md
```

---

## ✅ Prerequisites

- GitHub, AWS, and DockerHub accounts
- Installed locally:
  - `git`
  - `node` (v18+)
  - `docker`
  - `aws-cli`
  - `terraform`
  - `kubectl`
  - `eksctl`
  - `helm`

---

## ⚙️ Setup Instructions

### 1. Clone and run locally

```bash
git clone https://github.com/<your-username>/Trend.git
cd Trend
npm install
npm start          # runs on http://localhost:3000
```

### 2. Build and test the Docker image

```bash
docker build -t trend-app:latest .
docker run -d -p 3000:3000 --name trend-app-test trend-app:latest
```

### 3. Push the image to DockerHub

```bash
docker login
docker tag trend-app:latest <your-dockerhub-username>/trend-app:latest
docker push <your-dockerhub-username>/trend-app:latest
```

### 4. Provision AWS infrastructure with Terraform

```bash
cd terraform
terraform init
terraform plan
terraform apply
```

This creates a VPC, IAM role, and an EC2 instance with Jenkins pre-installed.

### 5. Configure Jenkins

1. Visit `http://<ec2-public-ip>:8080` and unlock Jenkins.
2. Install plugins: **Docker Pipeline**, **Git**, **Kubernetes CLI**, **Pipeline**.
3. Add credentials for GitHub and DockerHub under **Manage Jenkins → Credentials**.
4. Add a GitHub webhook pointing to `http://<ec2-public-ip>:8080/github-webhook/`.

### 6. Create the EKS cluster

```bash
eksctl create cluster --name trend-cluster --region us-east-1 \
  --nodegroup-name trend-nodes --node-type t3.medium \
  --nodes 2 --nodes-min 1 --nodes-max 3 --managed

aws eks --region us-east-1 update-kubeconfig --name trend-cluster
kubectl get nodes
```

### 7. Deploy to Kubernetes

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc trend-app-service   # note the EXTERNAL-IP
```

### 8. Set up the Jenkins pipeline

Create a Pipeline job in Jenkins pointing at this repo's `Jenkinsfile`, with the GitHub webhook trigger enabled.

From this point on, every `git push` to `main` automatically builds, pushes, and deploys the app.

### 9. Monitoring

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts

kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring

kubectl port-forward -n monitoring svc/prometheus-grafana 3001:80
```

Open Grafana at `http://localhost:3001`.

---

## 🔄 Pipeline Explanation

The `Jenkinsfile` defines a declarative pipeline with four stages:

| Stage | Description |
|-------|-------------|
| 1. Checkout | Pulls the latest code from GitHub |
| 2. Build Docker Image | Builds the app into a Docker image |
| 3. Push to DockerHub | Logs in and pushes the image with the `latest` tag |
| 4. Deploy to Kubernetes | Applies the manifests in `k8s/` and restarts the deployment so pods pull the new image |

The pipeline is triggered automatically by a **GitHub webhook** on every push to `main`.

---

## 🌐 Live Deployment

| Item | Value |
|------|-------|
| **Application URL** | `[http://<LoadBalancer external IP or DNS>](http://ad65ead04d5334bbda140538f13807f9-1744839119.ap-south-1.elb.amazonaws.com/]` |
| **LoadBalancer ARN** | `[ad65ead04d5334bbda140538f13807f9-1744839119.ap-south-1.elb.amazonaws.com]` |

---

## 📸 Screenshot Evidence

Each stage below is documented with a screenshot proving it worked end-to-end.

### 1. App Running Locally
Browser at `localhost:3000` showing the React app loaded, alongside the terminal running `npm start`.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/111a2438-da4a-4073-8806-3360e120d45b" />

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/5ab271e1-190e-4820-b4ae-8d721e822d69" />



### 2. Docker Build Success
Terminal showing `docker build -t trend-app:latest .` completing without errors.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/e30e350b-73e5-40e5-9719-6143c3d2b21b" />



### 3. Docker Container Running
Terminal output of `docker ps` showing the `trend-app-test` container as `Up`, and the browser at `localhost:3000` showing the app served from the container.

<img width="932" height="105" alt="image" src="https://github.com/user-attachments/assets/617649db-c5c6-491e-bf98-889e2a641559" />

<img width="950" height="130" alt="image" src="https://github.com/user-attachments/assets/bd608795-3e1a-434d-bb27-5d1cc39c53c8" />


<img width="1029" height="522" alt="image" src="https://github.com/user-attachments/assets/c57dc1ea-4ccc-4f92-a251-97e4ca592016" />


### 5. GitHub Repository
The repo's main page on GitHub showing all project files.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/2735d7b6-7edb-4192-9ca4-31bff81c11c7" />


### 6. Terraform Apply Success
Terminal showing the end of `terraform apply` output — the resource creation summary.

<img width="800" height="150" alt="image" src="https://github.com/user-attachments/assets/76440436-b666-44ce-8d36-01c3fdb9af68" />


### 7. AWS Console — Infrastructure Created
EC2 dashboard showing the running Jenkins instance, and the VPC dashboard showing the created VPC.
 
<img width="964" height="460" alt="image" src="https://github.com/user-attachments/assets/6ef057d5-4edb-4968-a398-40d2e52bc06f" />



### 8. Jenkins Unlocked and Dashboard
Jenkins login/unlock screen, proving the initial setup was completed.

<img width="1910" height="965" alt="image" src="https://github.com/user-attachments/assets/98932bc2-f39d-4822-a379-6078dd758bd5" />


### 9. GitHub Webhook Configured
GitHub repo → **Settings → Webhooks** page showing the webhook URL and a green checkmark for a successful recent delivery.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/79592ef8-ab86-4c78-a481-120503237063" />


### 10. EKS Cluster Running
Terminal output of `kubectl get nodes` showing node(s) in `Ready` status, and the AWS Console → EKS page showing the cluster status as `Active`.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/0a584b1c-853f-42a1-9db8-66ac41729c51" />


### 11. Kubernetes Deployment Applied
Terminal output of `kubectl get pods` showing pods `Running`, and `kubectl get svc trend-app-service` showing `TYPE: LoadBalancer` with an `EXTERNAL-IP`.

<img width="1032" height="428" alt="image" src="https://github.com/user-attachments/assets/4d8477b4-aa74-4e01-b64c-72d875600fd2" />

<img width="1911" height="911" alt="image" src="https://github.com/user-attachments/assets/d1d7eab3-117b-44fa-b38d-88f9122e2a74" />



### 12. Jenkins Pipeline Build
Jenkins pipeline job page showing a successful build in the history, the stage view (Checkout → Build → Push → Deploy, all green), and the console output of a build.
	
<img width="891" height="600" alt="image" src="https://github.com/user-attachments/assets/49330758-d3bf-42fe-9c8b-d1d6a3724a6c" />


### 13. Auto-Trigger Proof (Real CI/CD)
A `git push` in the terminal, immediately followed by Jenkins showing a new build auto-started.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/a462638f-c8bb-4c46-9edd-11b06b920f5a" />

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/bab680d7-b721-4c58-9143-67cdeae96b48" />


### 14. App Live on the Internet
Browser showing the app loaded from the LoadBalancer's external URL.

<img width="500" height="250" alt="image" src="https://github.com/user-attachments/assets/2ce65bd0-525c-4228-a3d5-0a1bc30b7f00" />


### 15. LoadBalancer ARN
AWS Console → EC2 → Load Balancers, showing the load balancer with its ARN visible.

<img width="750" height="500" alt="image" src="https://github.com/user-attachments/assets/c9920851-d3f8-4b31-ad14-7c77360a7233" />


### 16. Monitoring Dashboard
Terminal showing `kubectl get pods -n monitoring` with Prometheus/Grafana pods `Running`, and the Grafana dashboard in the browser showing live cluster/pod metrics (CPU, memory, pod count).


<img width="965" height="3131" alt="image" src="https://github.com/user-attachments/assets/f5c41a2e-c5c3-46c5-89b7-f1ac058a2d8d" />


---


