pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '519763206721.dkr.ecr.ap-south-1.amazonaws.com'
    }

    stages {

        stage('Clone Repo') {
            steps {
                git 'https://github.com/yogeshvshinde/StreamingApp.git'
            }
        }

        stage('Build Images') {
            steps {
                sh 'docker build -t admin-service ./backend/adminService'
                sh 'docker build -t auth-service ./backend/authService'
                sh 'docker build -t chat-service ./backend/chatService'
                sh 'docker build -t streaming-service ./backend/streamingService'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                docker tag admin-service:latest $ECR_REGISTRY/admin-service:latest
                docker tag auth-service:latest $ECR_REGISTRY/auth-service:latest
                docker tag chat-service:latest $ECR_REGISTRY/chat-service:latest
                docker tag streaming-service:latest $ECR_REGISTRY/streaming-service:latest

                docker push $ECR_REGISTRY/admin-service:latest
                docker push $ECR_REGISTRY/auth-service:latest
                docker push $ECR_REGISTRY/chat-service:latest
                docker push $ECR_REGISTRY/streaming-service:latest
                '''
            }
        }
    }
}