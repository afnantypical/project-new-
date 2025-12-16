pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'syedafnan9148'
        DOCKERHUB_CREDENTIALS = 'docker-hub-cred' // replace with your Jenkins Docker Hub credentials ID
        EC2_CREDENTIALS = 'ubuntu' // replace with your Jenkins SSH key credential ID
        EC2_HOST = '3.235.141.216'
        BACKEND_IMAGE = "${DOCKERHUB_USER}/project-new-backend:latest"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/project-new-frontend:latest"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh """
                        docker build -t ${BACKEND_IMAGE} .
                    """
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh """
                        docker build -t ${FRONTEND_IMAGE} .
                    """
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", passwordVariable: 'DOCKERHUB_PSW', usernameVariable: 'DOCKERHUB_USR')]) {
                    sh """
                        echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin
                        docker push ${BACKEND_IMAGE}
                        docker push ${FRONTEND_IMAGE}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(credentials: ["${EC2_CREDENTIALS}"]) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${EC2_HOST} '
                            set -e
                            echo "Pulling latest Docker images..."
                            docker pull ${BACKEND_IMAGE}
                            docker pull ${FRONTEND_IMAGE}

                            echo "Stopping old containers if running..."
                            docker stop backend frontend || true
                            docker rename backend backend_old || true
                            docker rename frontend frontend_old || true

                            echo "Running new containers..."
                            docker run -d --restart unless-stopped --name backend -p 3000:3000 ${BACKEND_IMAGE}
                            docker run -d --restart unless-stopped --name frontend -p 4200:4200 ${FRONTEND_IMAGE}

                            echo "Waiting for services to start..."
                            sleep 10

                            echo "Checking health..."
                            if ! curl -f http://localhost:3000/health; then
                                echo "Backend failed health check. Rolling back..."
                                docker rm -f backend
                                docker rename backend_old backend
                            fi

                            if ! curl -f http://localhost:4200/; then
                                echo "Frontend failed health check. Rolling back..."
                                docker rm -f frontend
                                docker rename frontend_old frontend
                            fi

                            echo "Cleaning up old containers..."
                            docker rm -f backend_old frontend_old || true
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
