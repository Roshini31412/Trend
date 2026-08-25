**Trend — React App CI/CD Deployment on AWS EKS**
Deploying a React application to a production-ready state using Docker, Terraform, Jenkins, AWS EKS, and Kubernetes, with automated CI/CD and open-source monitoring.


**Overview**
This project takes a React application from source code to a fully automated production deployment:
GitHub push → Jenkins (build & test) → Docker image → DockerHub
   → Kubernetes (AWS EKS) → LoadBalancer → Live application
                     ↑
        Prometheus + Grafana (monitoring)

Every push to main automatically triggers Jenkins to build a new Docker image, push it to DockerHub, and roll it out to the Kubernetes cluster — no manual steps required after the initial setup.

Tech stack
Layer	Tool
Application	React
Containerization	Docker
Infrastructure as Code	Terraform
CI/CD	Jenkins
Image Registry	DockerHub
Orchestration	Kubernetes on AWS EKS
Monitoring	Prometheus + Grafana
**
**Project structure****
Trend/
├── Dockerfile              # Multi-stage build for the React app
├── .dockerignore
├── .gitignore
├── Jenkinsfile              # Declarative CI/CD pipeline
├── terraform/
│   └── main.tf              # VPC, IAM, EC2 + Jenkins provisioning
├── k8s/
│   ├── deployment.yaml       # Kubernetes Deployment (2 replicas)
│   └── service.yaml          # Kubernetes LoadBalancer Service
├── docs/
│   └── screenshots/          # Screenshots referenced below
└── README.md


**Prerequisites**
	• GitHub, AWS, and DockerHub accounts
	• Installed locally: git, node (v18+), docker, aws-cli, terraform, kubectl, eksctl, helm

**Setup instructions**

**1. Clone and run locally**
git clone https://github.com/<your-username>/Trend.git
cd Trend
npm install
npm start          # runs on http://localhost:3000

**2. Build and test the Docker image**
docker build -t trend-app:latest .
docker run -d -p 3000:3000 --name trend-app-test trend-app:latest

**3. Push the image to DockerHub**

docker login
docker tag trend-app:latest <your-dockerhub-username>/trend-app:latest
docker push <your-dockerhub-username>/trend-app:latest

**4. Provision AWS infrastructure with Terraform**

cd terraform
terraform init
terraform plan
terraform apply
This creates a VPC, IAM role, and an EC2 instance with Jenkins pre-installed.

**5. Configure Jenkins**

	1. Visit http://<ec2-public-ip>:8080 and unlock Jenkins.
	2. Install plugins: Docker Pipeline, Git, Kubernetes CLI, Pipeline.
	3. Add credentials for GitHub and DockerHub under Manage Jenkins → Credentials.
	4. Add a GitHub webhook pointing to http://<ec2-public-ip>:8080/github-webhook/.
  
**6. Create the EKS cluster**

  1. eksctl create cluster --name trend-cluster --region us-east-1 \ --nodegroup-name trend-nodes --node-type t3.medium \
      --nodes 2 --nodes-min 1 --nodes-max 3 --managed
  2. aws eks --region us-east-1 update-kubeconfig --name trend-cluster
  3. kubectl get nodes
     
**7. Deploy to Kubernetes**
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl get svc trend-app-service   # note the EXTERNAL-IP

**8. Set up the Jenkins pipeline**
Create a Pipeline job in Jenkins pointing at this repo's Jenkinsfile, with the GitHub webhook trigger enabled. From this point on, every git push to main automatically builds, pushes, and deploys the app.

**9. Monitoring**

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
kubectl create namespace monitoring
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
kubectl port-forward -n monitoring svc/prometheus-grafana 3001:80
Open Grafana at http://localhost:3001.

**Pipeline explanation**
The Jenkinsfile defines a declarative pipeline with four stages:
	1. Checkout – pulls the latest code from GitHub.
	2. Build Docker Image – builds the app into a Docker image.
	3. Push to DockerHub – logs in and pushes the image with the latest tag.
	4. Deploy to Kubernetes – applies the manifests in k8s/ and restarts the deployment so pods pull the new image.
The pipeline is triggered automatically by a GitHub webhook on every push to main.

Live deployment
	• Application URL: http://<LoadBalancer external IP or DNS>


Screenshots
Stage	Screenshot
App running locally	docs/screenshots/local-app.png
Docker image running	docs/screenshots/docker-run.png
DockerHub repository	docs/screenshots/dockerhub.png
Terraform apply output	docs/screenshots/terraform-apply.png
Jenkins pipeline success	docs/screenshots/jenkins-pipeline.png
kubectl get pods / get svc	docs/screenshots/kubectl-output.png
App live via LoadBalancer	docs/screenshots/live-app.png
Grafana dashboard	docs/screenshots/grafana.png


