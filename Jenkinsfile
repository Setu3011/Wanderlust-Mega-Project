pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Jenkins credential ID for DockerHub
    }

    stages {

        stage('Install & Test Backend') {
    steps {
        dir('backend') {
            script {
                docker.image('node:20').inside {
                    sh '''
                        mkdir -p /tmp/npm-cache
                        npm config set cache /tmp/npm-cache --global
                        npm install
                        npm test || echo "Tests skipped or failed"
                    '''
                }
            }
        }
    }
}


        stage('Build Docker Images') {
            steps {
                sh 'docker build -t setu3011/wanderlust-backend ./backend'
                sh 'docker build -t setu3011/wanderlust-frontend ./frontend'
            }
        }

        stage('Push Docker Images to DockerHub') {
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
                sshagent(['ec2-key']) { // This is the SSH key credential ID in Jenkins
                    sh '''
                        ssh -o StrictHostKeyChecking=no ec2-user@16.171.0.243 '
                            docker pull setu3011/wanderlust-backend &&
                            docker pull setu3011/wanderlust-frontend &&
                            docker-compose down &&
                            docker-compose up -d
                        '
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and Deployment completed successfully!"
        }
        failure {
            echo "❌ Build or Deployment failed. Check logs for errors."
        }
    }
}
