# Microservice Deployment in Kubernetes with Jenkins CI/CD

This repository demonstrates the process of deploying a microservice application in Kubernetes using Jenkins for Continuous Integration and Continuous Deployment (CI/CD), Docker for containerization, and AWS for hosting the infrastructure.

## Prerequisites

- **Docker**: Ensure Docker is installed for containerizing the application.
- **Kubernetes**: Set up a Kubernetes cluster (e.g., using AWS EKS).
- **Jenkins**: Set up Jenkins with a multibranch pipeline.
- **AWS CLI**: AWS CLI to interact with AWS services.
- **IAM Roles**: Ensure proper permissions for Jenkins to interact with AWS.

## Architecture

- **Microservices**: Application split into multiple services.
- **Jenkins**: Automates the CI/CD pipeline, with multibranch support.
- **Docker**: Used to create container images of the microservices.
- **Kubernetes (EKS)**: Manages deployment and scaling of containers.
- **AWS**: Hosts the Kubernetes cluster, Jenkins, and container images.

## Steps

### 1. Dockerize the Application
- Create a `Dockerfile` for each microservice.
- Build and push images to Docker Hub or AWS ECR.

### 2. Jenkins Setup
- Install Jenkins on an EC2 instance or use AWS Jenkins service.
- Set up **multibranch pipelines** for each feature branch.
- Configure Jenkins to pull from the GitHub repository.

### 3. Configure Jenkins Pipeline
- **Jenkinsfile**: Define the pipeline steps for build, test, and deploy.
- Use Jenkins plugins for Docker, Kubernetes, and AWS integration.

### 4. Build & Push Docker Images
- Jenkins builds Docker images for each commit.
- Push images to Docker Hub or AWS ECR for storage.

### 5. Deploy to Kubernetes
- Use `kubectl` or Helm for deploying the Docker containers to AWS EKS.
- Automate the deployment process in Jenkins.

### 6. CI/CD Workflow
- Jenkins triggers on each commit or pull request to build and test the microservice.
- After successful testing, Jenkins deploys the service to Kubernetes.

### 7. AWS EKS Setup
- Create an EKS cluster in AWS.
- Configure `kubectl` to interact with your cluster.

## Example Jenkinsfile

```groovy
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "your-ecr-repository/your-image"
        AWS_REGION = "us-west-2"
    }
    stages {
        stage('Build') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry("https://$AWS_ECR_URL", 'aws-credentials') {
                        docker.image(DOCKER_IMAGE).push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }
    }
}
Conclusion
This setup automates the CI/CD pipeline for microservices in a Kubernetes environment using Jenkins, Docker, AWS, and multibranch pipeline support. Jenkins handles building, testing, and deploying to Kubernetes for a seamless deployment process.

