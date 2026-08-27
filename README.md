**Trend — React App CI/CD Deployment on AWS EKS**
Deploying a React- application to a production-ready state using Docker, Terraform, Jenkins, AWS EKS, and Kubernetes, with automated CI/CD and open-source monitoring.


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


<img width="526" height="345" alt="Screenshot 2026-08-26 073314" src="https://github.com/user-attachments/assets/c8ad5879-56d3-47cd-8f8b-9874ac23a4b6" />


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

**Live deployment**
	• Application URL: http://<LoadBalancer external IP or DNS>


**Screenshots**
Stage	Screenshot
App running locally	:
Here's the full list, in order, with exactly what should be visible in each shot so it's obvious to anyone grading it that each stage actually worked.
1. App running locally
	• Browser at localhost:3000 showing the React app loaded

	• Terminal showing npm start output alongside it:

	• 
2. Docker build success
	• Terminal showing docker build -t trend-app:latest . completing without errors.
	• 
	

3. Docker container running
	• Terminal output of docker ps showing the trend-app-test container as Up
	
	• 
	• 
   Browser at localhost:3000 showing the app served from the container






5. GitHub repository
	• My repo's main page on GitHub showing all files:


	• 
6. Terraform apply success
	• Terminal showing the end of terraform apply output: resource creation summary (Apply complete! Resources: X added) and the jenkins_public_ip output value

7. AWS Console — infrastructure created
	• EC2 dashboard showing the running Jenkins instance
	• 
	• VPC dashboard showing the created VPC:
	• 
8. Jenkins unlocked and dashboard
	• Jenkins login/unlock screen (proves you set it up)

	• Jenkins dashboard showing installed plugins or the plugin manager with Docker Pipeline, Git, Kubernetes CLI, Pipeline checked
9. GitHub webhook configured
	• GitHub repo → Settings → Webhooks page showing the webhook URL and a green checkmark (recent delivery successful)
10. EKS cluster running
	• Terminal output of kubectl get nodes showing node(s) in Ready status

	• AWS Console → EKS → your cluster showing status Active


11. Kubernetes deployment applied
	• Terminal output of kubectl get pods showing your pods in Running status
	• 
	• Terminal output of kubectl get svc trend-app-service showing TYPE: LoadBalancer and an EXTERNAL-IP



12. Jenkins pipeline build
	• Jenkins pipeline job page showing a green/successful build in the build history
	
	• The pipeline stage view (Checkout → Build → Push → Deploy, all green)

	• Console output of one build (optional, shows the docker push and kubectl apply happening)

13. Auto-trigger proof (important one — shows real CI/CD, not manual)
	• A git push in your terminal

	• Immediately followed by Jenkins showing a new build auto-started (timestamp matching)

14. App live on the internet
	• Browser showing the app loaded from the LoadBalancer's external URL (http://ad65ead04d5334bbda140538f13807f9-1744839119.ap-south-1.elb.amazonaws.com/) —


	• 
	• 
15. LoadBalancer ARN
	• AWS Console → EC2 → Load Balancers, showing the load balancer with its ARN visible .

	• 
	• 
16. Monitoring dashboard
	• Terminal showing kubectl get pods -n monitoring with Prometheus/Grafana pods Running
	• Grafana dashboard in browser showing live cluster/pod metrics (CPU, memory, pod count)
<img width="1172" height="7466" alt="image" src="https://github.com/user-attachments/assets/ea186360-6b3f-4923-aa3c-378ea25e9f34" />


