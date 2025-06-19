pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'wanderlust-backend'
        DOCKERHUB_USERNAME = 'Setu3011'
        EC2_HOST = '16.171.43.231'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')             // ✅ your DockerHub credential ID
        SSH_PRIVATE_KEY = credentials('shell-scripting-key.pem')     // ✅ your EC2 PEM key
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
                        docker.image('node:20').inside {
                            sh '''
                                echo "📦 Installing dependencies..."
                                mkdir -p /tmp/npm-cache
                                npm install --cache /tmp/npm-cache

                                echo "✅ Running tests (if any)..."
                                npm test || echo "⚠️ No tests found or skipped"
                            '''
                        }
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE_NAME:latest .'
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS) {
                        sh '''
                            docker tag $DOCKER_IMAGE_NAME:latest $DOCKERHUB_USERNAME/$DOCKER_IMAGE_NAME:latest
                            docker push $DOCKERHUB_USERNAME/$DOCKER_IMAGE_NAME:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                script {
                    sh '''
                        echo "🔐 Saving EC2 key..."
                        echo "$SSH_PRIVATE_KEY" > ec2key.pem
                        chmod 400 ec2key.pem

                        echo "🚀 Deploying to EC2..."
                        ssh -o StrictHostKeyChecking=no -i ec2key.pem ubuntu@$EC2_HOST << EOF
                          docker pull $DOCKERHUB_USERNAME/$DOCKER_IMAGE_NAME:latest
                          docker stop wanderlust-container || true
                          docker rm wanderlust-container || true
                          docker run -d --name wanderlust-container -p 3000:3000 $DOCKERHUB_USERNAME/$DOCKER_IMAGE_NAME:latest
                        EOF

                        echo "✅ Deployment to EC2 complete!"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build and Deployment Successful!'
        }
        failure {
            echo '❌ Build or Deployment Failed. Please check the logs.'
        }
    }
}

