pipeline {
    agent any

    environment {
        BACKEND_IMAGE = 'syedafnan9148/project-new-backend:latest'
        FRONTEND_IMAGE = 'syedafnan9148/project-new-frontend:latest'
        APP_SERVER = 'ubuntu@3.235.141.216' // replace with your server
        DEPLOY_DIR = '/home/ubuntu/project' // path on your server
    }

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/afnantypical/project-new-.git'
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh 'docker build -t $BACKEND_IMAGE .'
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh 'docker build -t $FRONTEND_IMAGE .'
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', 
                                                  usernameVariable: 'DOCKERHUB_USER', 
                                                  passwordVariable: 'DOCKERHUB_PSW')]) {
                    sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USER --password-stdin'
                    sh 'docker push $BACKEND_IMAGE'
                    sh 'docker push $FRONTEND_IMAGE'
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(['ubuntu']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no $APP_SERVER '
                        docker pull $BACKEND_IMAGE &&
                        docker pull $FRONTEND_IMAGE &&
                        docker stop backend || true &&
                        docker rm backend || true &&
                        docker run -d --name backend -p 3000:3000 $BACKEND_IMAGE &&
                        docker stop frontend || true &&
                        docker rm frontend || true &&
                        docker run -d --name frontend -p 4200:4200 $FRONTEND_IMAGE
                        '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
