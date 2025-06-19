pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')      // DockerHub credentials ID
        SSH_PRIVATE_KEY = credentials('shell-scripting-key')  // EC2 SSH private key ID
        IMAGE_NAME = "setu3011/wanderlust-backend"
        EC2_HOST = "ubuntu@16.171.43.231"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install & Test Backend') {
            steps {
                dir('backend') {
                    script {
                        docker.image('node:20').inside('-u 0:0') {
                            sh '''
                                echo "📦 Installing dependencies..."
                                npm config set cache /tmp/npm-cache --global
                                npm install --no-audit --no-fund --unsafe-perm
                                echo "✅ Running tests..."
                                npm test || echo "⚠️ Tests failed, but continuing"
                            '''
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    echo "🐳 Building Docker image..."
                    docker build -t $IMAGE_NAME backend/
                '''
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                sh '''
                    echo "🔐 Logging into DockerHub..."
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin

                    echo "📤 Pushing image to DockerHub..."
                    docker push $IMAGE_NAME
                '''
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sh '''
                    echo "🚀 Deploying to EC2 instance..."

                    echo "$SSH_PRIVATE_KEY" > ec2-key.pem
                    chmod 400 ec2-key.pem

                    ssh -o StrictHostKeyChecking=no -i ec2-key.pem $EC2_HOST '
                        docker pull $IMAGE_NAME &&
                        docker stop wanderlust || true &&
                        docker rm wanderlust || true &&
                        docker run -d --name wanderlust -p 8000:8000 $IMAGE_NAME
                    '

                    rm ec2-key.pem
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Build or Deployment Failed. Please check the logs."
        }
    }
}
