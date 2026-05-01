pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '519763206721.dkr.ecr.ap-south-1.amazonaws.com'
        CLUSTER_NAME = 'mern-cluster'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Clone Repo') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/yogeshvshinde/StreamingApp.git'
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                echo "Building Docker images..."

                docker build -t auth-service ./backend/authService
                docker build -t streaming-service ./backend/streamingService
                docker build -t admin-service ./backend/adminService
                docker build -t chat-service ./backend/chatService
                docker build -t frontend ./frontend
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                    echo "Logging into ECR..."
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh '''
                echo "Pushing images to ECR..."

                for service in auth-service streaming-service admin-service chat-service frontend
                do
                  docker tag $service:latest $ECR_REGISTRY/$service:latest
                  docker push $ECR_REGISTRY/$service:latest
                done
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                withAWS(credentials: 'aws-creds', region: "${AWS_REGION}") {
                    sh '''
                    echo "Deploying to EKS..."

                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                    kubectl apply -f k8s/

                    kubectl get pods
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}