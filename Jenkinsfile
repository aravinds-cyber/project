pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "public.ecr.aws/h0k7k9z9/api"
        ECR_REPO = "public.ecr.aws/h0k7k9z9/frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aravinds-cyber/project.git'
            }
        }

        stage('Build & Push API Image') {
            steps {
                sh '''
                cd api
                docker build -t $ECR_REPO/api:latest .
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                docker push $ECR_REPO/api:latest
                '''
            }
        }

        stage('Build & Push Frontend Image') {
            steps {
                sh '''
                cd frontend
                docker build -t $ECR_REPO/frontend:latest .
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                docker push $ECR_REPO/frontend:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                aws eks update-kubeconfig --region $AWS_REGION --name kubernetes
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
