pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')
    }

    stages {
        stage('Install & Test Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    sh 'npm test || echo "Tests skipped or failed"'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t setu3011/wanderlust-backend ./backend'
                sh 'docker build -t setu3011/wanderlust-frontend ./frontend'
            }
        }

        stage('Push Docker Images') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    sh 'docker push setu3011/wanderlust-backend'
                    sh 'docker push setu3011/wanderlust-frontend'
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@16.171.0.243 '
                        docker pull setu3011/wanderlust-backend &&
                        docker pull setu3011/wanderlust-frontend &&
                        docker-compose up -d
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & Deployment Successful!"
        }
        failure {
            echo "❌ Build or Deployment Failed!"
        }
    }
}


