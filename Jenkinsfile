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
                sh 'docker build -t frontend ./frontend'
                sh 'docker build -t backend ./backend'
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
                docker tag frontend:latest $ECR_REGISTRY/frontend:latest
                docker tag backend:latest $ECR_REGISTRY/backend:latest

                docker push $ECR_REGISTRY/frontend:latest
                docker push $ECR_REGISTRY/backend:latest
                '''
            }
        }
    }
}