pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '519763206721.dkr.ecr.ap-south-1.amazonaws.com'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git credentialsId: 'github-creds',
                    git branch: 'main', url: 'https://github.com/yogeshvshinde/StreamingApp.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                docker build -t auth-service ./backend/authService
                docker build -t streaming-service ./backend/streamingService
                docker build -t admin-service ./backend/adminService
                docker build -t chat-service ./backend/chatService
                docker build -t frontend ./frontend
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                    sh '''
                    aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY

                    for service in auth-service streaming-service admin-service chat-service frontend
                    do
                      docker tag $service:latest $ECR_REGISTRY/$service:latest
                      docker push $ECR_REGISTRY/$service:latest
                    done
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'ap-south-1') {
                    sh '''
                    aws eks update-kubeconfig --region $AWS_REGION --name mern-cluster
                    kubectl apply -f k8s/
                    '''
                }
            }
        }
    }
}