pipeline {
    agent any

    environment {
        AWS_REGION        = "us-east-1" 
        ECR_REPO_FRONTEND = "public.ecr.aws/h0k7k9z9/frontend"
        ECR_REPO_API      = "public.ecr.aws/h0k7k9z9/api"
        IMAGE_TAG         = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/aravinds-cyber/project.git'
            }
        }

        stage('AWS Login & Docker Build/Push') {
            steps {
                withAWS(credentials: 'aws-credentials-id-2', region: "${AWS_REGION}") {
                    echo "Logging in to ECR..."
                    sh """
                        aws ecr-public get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin public.ecr.aws
                    """

                    echo "Building and pushing frontend image..."
                    sh """
                        docker build -t ${ECR_REPO_FRONTEND}:${IMAGE_TAG} ./frontend
                        docker push ${ECR_REPO_FRONTEND}:${IMAGE_TAG}
                    """

                    echo "Building and pushing API image..."
                    sh """
                        docker build -t ${ECR_REPO_API}:${IMAGE_TAG} ./api
                        docker push ${ECR_REPO_API}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([file(credentialsId: 'eks-kubeconfig', variable: 'KUBECONFIG')]) {
                    echo "Checking EKS nodes..."
                    sh 'kubectl get nodes'

                    echo "Applying Kubernetes manifests..."
                    sh """
                        kubectl apply -f k8s/01-namespace.yaml --validate=false
                        kubectl apply -f k8s/02-configmap-frontend.yaml --validate=false
                        kubectl apply -f k8s/03-deployment-api.yaml --validate=false
                        kubectl apply -f k8s/04-service-api.yaml --validate=false
                        kubectl apply -f k8s/05-deployment-frontend.yaml --validate=false
                        kubectl apply -f k8s/06-service-frontend.yaml --validate=false
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}











