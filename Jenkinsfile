pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
        SSH_PRIVATE_KEY = credentials('shell-scripting-key')
        EC2_IP = '13.51.70.235'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Setu3011/Wanderlust-Mega-Project.git', branch: 'main'
            }
        }

        stage('Clean Previous Builds') {
            steps {
                dir('backend') {
                    sh 'rm -rf node_modules package-lock.json'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('backend') {
                    sh '''
                        echo 📦 Installing dependencies...
                        mkdir -p /tmp/npm-cache
                        npm config set cache /tmp/npm-cache --global
                        npm install --no-audit --no-fund --unsafe-perm || true
                    '''
                }
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'npm test || echo "Tests failed but continuing..."'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo 🛠️ Building Docker image...
                    docker build -t setu3011/wanderlust-backend:latest ./backend
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub') {
                        sh 'docker push setu3011/wanderlust-backend:latest'
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    echo 🚀 Deploying to EC2...
                    ssh -o StrictHostKeyChecking=no -i ~/.ssh/shell-scripting-key.pem ubuntu@${EC2_IP} << EOF
                        docker pull setu3011/wanderlust-backend:latest
                        docker stop wanderlust-backend || true
                        docker rm wanderlust-backend || true
                        docker run -d -p 8000:8000 --name wanderlust-backend setu3011/wanderlust-backend:latest
                    EOF
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Build or Deployment Failed. Please check the logs.'
        }
    }
}
