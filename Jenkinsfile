pipeline {
    agent any

    environment {
        AWS_REGION        = "us-east-1" // Change to your AWS region
        ECR_REPO_FRONTEND = "public.ecr.aws/h0k7k9z9/frontend"
        ECR_REPO_API      = "public.ecr.aws/h0k7k9z9/api"
        EKS_CLUSTER       = "kubernetes" // Change to your cluster name
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aravinds-cyber/project.git'
            }
        }

        stage('Build & Push Frontend') {
            steps {
                sh '''
                echo "Logging in to ECR for frontend..."
                aws ecr-public get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin public.ecr.aws

                echo "Building frontend image..."
                docker build -t $ECR_REPO_FRONTEND:latest ./frontend

                echo "Pushing frontend image..."
                docker push $ECR_REPO_FRONTEND:latest
                '''
            }
        }

        stage('Build & Push API') {
            steps {
                sh '''
                echo "Logging in to ECR for API..."
                aws ecr-public get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin public.ecr.aws

                echo "Building API image..."
                docker build -t $ECR_REPO_API:latest ./api

                echo "Pushing API image..."
                docker push $ECR_REPO_API:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                echo "Updating kubeconfig..."
                aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER

                echo "Applying Kubernetes manifests..."
                kubectl apply -f k8s/01-namespace.yaml
                kubectl apply -f k8s/02-configmap-frontend.yaml
                kubectl apply -f k8s/03-deployment-api.yaml
                kubectl apply -f k8s/04-service-api.yaml
                kubectl apply -f k8s/05-deployment-frontend.yaml
                kubectl apply -f k8s/06-service-frontend.yaml
                '''
            }
        }
    }
}
