pipeline {
    agent any

    environment {
        DOCKERHUB_BACKEND  = "syedafnan9148/project-new-backend"
        DOCKERHUB_FRONTEND = "syedafnan9148/project-new-frontend"
        SSH_HOST = "3.235.141.216"
        SSH_USER = "ubuntu"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_BACKEND}:latest ./backend'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_FRONTEND}:latest ./frontend'
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKERHUB_USR',
                        passwordVariable: 'DOCKERHUB_PSW'
                    )
                ]) {
                    sh '''
                        set -e
                        echo "$DOCKERHUB_PSW" | docker login -u "$DOCKERHUB_USR" --password-stdin
                        docker push syedafnan9148/project-new-backend:latest
                        docker push syedafnan9148/project-new-frontend:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sshagent(['app-ec2-ssh']) {
                    sh '''
                        set -e
                        ssh -o StrictHostKeyChecking=no ubuntu@3.235.141.216 << 'EOF'

                        docker pull syedafnan9148/project-new-backend:latest
                        docker pull syedafnan9148/project-new-frontend:latest

                        docker stop backend || true
                        docker stop frontend || true

                        docker rm backend || true
                        docker rm frontend || true

                        docker run -d --name backend -p 3000:3000 syedafnan9148/project-new-backend:latest
                        docker run -d --name frontend -p 4200:4200 syedafnan9148/project-new-frontend:latest

                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment completed successfully'
        }
        failure {
            echo '❌ Pipeline failed. Check Jenkins logs'
        }
    }
}
