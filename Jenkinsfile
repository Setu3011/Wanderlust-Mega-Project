pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SSH_PRIVATE_KEY = credentials('shell-scripting-key')
        EC2_IP = '13.51.194.96'
        IMAGE_NAME = 'wanderlust-backend'
        DOCKER_USER = 'Setu3011'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Setu3011/Wanderlust-Mega-Project.git'
            }
        }

        stage('Install & Test Backend') {
            steps {
                dir('backend') {
                    script {
                        docker.image('node:20-alpine').inside('-u root:root') {
                            sh '''
                                echo 📦 Installing dependencies...
                                rm -rf node_modules package-lock.json
                                npm cache clean --force
                                npm install --no-audit --no-fund --unsafe-perm || {
                                    echo "⚠️ npm install failed. Trying minimal install..."
                                    npm install --ignore-scripts
                                }
                            '''
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_USER/$IMAGE_NAME:latest ./backend'
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                withDockerRegistry([credentialsId: 'dockerhub']) {
                    sh 'docker push $DOCKER_USER/$IMAGE_NAME:latest'
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sh '''
                    echo "🔐 Connecting to EC2 and deploying..."
                    echo "$SSH_PRIVATE_KEY" > key.pem
                    chmod 400 key.pem

                    ssh -o StrictHostKeyChecking=no -i key.pem ec2-user@$EC2_IP << 'ENDSSH'
                        docker pull $DOCKER_USER/$IMAGE_NAME:latest
                        docker stop backend || true
                        docker rm backend || true
                        docker run -d --name backend -p 8000:8000 $DOCKER_USER/$IMAGE_NAME:latest
                    ENDSSH

                    rm -f key.pem
                '''
            }
        }
    }

    post {
        failure {
            echo "❌ Build or Deployment Failed. Please check the logs."
        }
        success {
            echo "✅ Successfully deployed to http://$EC2_IP:8000"
        }
    }
}
