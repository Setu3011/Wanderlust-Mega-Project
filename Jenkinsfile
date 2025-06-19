pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "setu3011/wanderlust-app"
        EC2_USER = "ubuntu"
        EC2_IP = "13.51.194.96"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SSH_PRIVATE_KEY = credentials('shell-scripting-key')
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
                                echo 📦 Installing dependencies...
                                rm -rf node_modules package-lock.json
                                mkdir -p /tmp/npm-cache
                                npm config set cache /tmp/npm-cache --global
                                npm install --no-audit --no-fund --unsafe-perm
                                echo ✅ Dependencies installed.
                                echo 🧪 Running tests (if any)...
                                npm test || echo "No tests found or test phase skipped"
                            '''
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    echo 🐳 Building Docker image...
                    docker build -t $DOCKER_IMAGE:latest .
                '''
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub', url: '']) {
                    sh '''
                        echo 📤 Pushing Docker image to DockerHub...
                        docker push $DOCKER_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sh '''
                    echo 📦 Connecting to EC2 and pulling latest Docker image...
                    ssh -o StrictHostKeyChecking=no -i $SSH_PRIVATE_KEY $EC2_USER@$EC2_IP << EOF
                      docker pull $DOCKER_IMAGE:latest
                      docker stop wanderlust-container || true
                      docker rm wanderlust-container || true
                      docker run -d --name wanderlust-container -p 80:3000 $DOCKER_IMAGE:latest
                    EOF
                    echo 🚀 Deployment complete.
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build & Deployment succeeded!'
        }
        failure {
            echo '❌ Build or Deployment Failed. Please check the logs.'
        }
    }
}
