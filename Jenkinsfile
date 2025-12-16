pipeline {
    agent any

    environment {
        // Docker Hub repositories
        DOCKERHUB_BACKEND  = "syedafnan9148/project-new-backend"
        DOCKERHUB_FRONTEND = "syedafnan9148/project-new-frontend"

        // EC2 App Server details (UPDATED)
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
                sh """
                    docker build -t ${DOCKERHUB_BACKEND}:latest \
                    ./project-new/backend
                """
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKERHUB_FRONTEND}:latest \
                    ./project-new/frontend
                """
            }
        }

        stage('Push Images to Docker Hub') {
            environment {
                DOCKERHUB = credentials('dockerhub-cred')
            }
            steps {
                sh """
                    echo "\$DOCKERHUB_PSW" | docker login \
                    -u "\$DOCKERHUB_USR" --password-stdin

                    docker push ${DOCKERHUB_BACKEND}:latest
                    docker push ${DOCKERHUB_FRONTEND}:latest
                """
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sshagent(credentials: ['app-ec2-ssh']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no \
                        ${SSH_USER}@${SSH_HOST} '
                            cd /opt/project-new &&
                            git pull &&
                            docker-compose pull &&
                            docker-compose up -d
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully"
        }
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs"
        }
    }
}
