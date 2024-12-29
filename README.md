README - Deploying Microservice Application with Jenkins, Docker, Kubernetes on AWS
This guide outlines how to deploy a microservice application using Jenkins for CI/CD, Docker for containerization, and Kubernetes on AWS.

Prerequisites
AWS Account
Jenkins Instance with Multibranch Pipeline plugin
Docker and kubectl installed
AWS EKS (Elastic Kubernetes Service) cluster
Architecture Overview
Microservices built with Node.js, Java, or Python.
Jenkins for continuous integration with a multibranch pipeline.
Docker for containerization.
AWS EKS for Kubernetes-based deployment.
1. Set Up Jenkins
Install Jenkins on an EC2 instance:
Follow this guide.
Install the following plugins in Jenkins:
Docker Pipeline
Kubernetes CLI
Multibranch Pipeline
2. Create Jenkins Multibranch Pipeline
Create a Multibranch Pipeline job in Jenkins.
Configure the GitHub repository (with webhook).
Define the pipeline in Jenkinsfile:
groovy
Copy code
pipeline {
    agent any
    environment { 
        REGISTRY = 'your-docker-registry' 
        IMAGE_NAME = 'microservice'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                script { docker.build("${REGISTRY}/${IMAGE_NAME}:latest") }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-credentials') {
                        docker.image("${REGISTRY}/${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    kubectl set image deployment/${IMAGE_NAME} ${IMAGE_NAME}=${REGISTRY}/${IMAGE_NAME}:latest
                }
            }
        }
    }
}
3. Dockerize the Application
Create a Dockerfile for your microservice (e.g., Node.js):

dockerfile
Copy code
FROM node:14
WORKDIR /app
COPY . .
RUN npm install
EXPOSE 3000
CMD ["node", "app.js"]
Build and test locally:

bash
Copy code
docker build -t your-docker-registry/microservice:latest .
docker run -p 3000:3000 your-docker-registry/microservice:latest
4. Create Kubernetes Cluster on AWS (EKS)
Install eksctl:
bash
Copy code
curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/v0.78.0/eksctl_Linux_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
Create the cluster:
bash
Copy code
eksctl create cluster --name your-cluster --region us-west-2 --nodegroup-name standard-workers --node-type t2.micro --nodes 3
Update kubectl:
bash
Copy code
aws eks --region us-west-2 update-kubeconfig --name your-cluster
5. Deploy Application to Kubernetes
Create a Kubernetes deployment.yaml:
yaml
Copy code
apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice
spec:
  replicas: 3
  selector:
    matchLabels:
      app: microservice
  template:
    metadata:
      labels:
        app: microservice
    spec:
      containers:
        - name: microservice
          image: your-docker-registry/microservice:latest
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: microservice
spec:
  selector:
    app: microservice
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
Apply to EKS:
bash
Copy code
kubectl apply -f deployment.yaml
6. CI/CD Flow
Code Commit: Developer pushes to GitHub/Bitbucket.
Jenkins Trigger: Jenkins pipeline is triggered.
Build & Push Docker Image: Jenkins builds and pushes to Docker registry.
Deploy to Kubernetes: Jenkins updates the Kubernetes deployment with the new image.
Troubleshooting
Jenkins Build Failures: Check Docker and Kubernetes credentials.
Kubernetes Deploy Failures: Ensure correct IAM roles and Kubernetes configuration.
