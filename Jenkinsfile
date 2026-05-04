pipeline {
    agent any

    triggers {
        githubPush()
    }

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
                sh '''
                echo "Logging into ECR..."
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REGISTRY
                '''
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

        stage('Configure EKS Access') {
            steps {
                sh '''
                echo "Configuring kubeconfig..."
                aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
                '''
            }
        }

        stage('Create Kubernetes Secrets') {
            steps {
                withCredentials([
                    string(credentialsId: 'jwt-secret', variable: 'JWT_SECRET'),
                    string(credentialsId: 'aws-access-key-id', variable: 'APP_AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key', variable: 'APP_AWS_SECRET_ACCESS_KEY'),
                    string(credentialsId: 's3-bucket-name', variable: 'AWS_S3_BUCKET')
                ]) {
                    sh '''
                    kubectl create secret generic streaming-app-secrets \
                      --from-literal=JWT_SECRET="$JWT_SECRET" \
                      --from-literal=AWS_REGION="$AWS_REGION" \
                      --from-literal=AWS_S3_BUCKET="$AWS_S3_BUCKET" \
                      --from-literal=AWS_ACCESS_KEY_ID="$APP_AWS_ACCESS_KEY_ID" \
                      --from-literal=AWS_SECRET_ACCESS_KEY="$APP_AWS_SECRET_ACCESS_KEY" \
                      --dry-run=client -o yaml | kubectl apply -f -
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                echo "Deploying to EKS..."
                kubectl apply -f k8s/

                for deployment in auth-service streaming-service admin-service chat-service frontend
                do
                  kubectl rollout restart deployment/$deployment
                  kubectl rollout status deployment/$deployment
                done
                '''
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
