pipeline {
    agent any

    environment {
        BACKEND_IMAGE  = "syedafnan9148/project-new-backend:latest"
        FRONTEND_IMAGE = "syedafnan9148/project-new-frontend:latest"
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git(
                    url: 'https://github.com/afnantypical/project-new-.git',
                    branch: 'main'
                )
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('backend') {
                    sh '''
                        docker build -t $BACKEND_IMAGE .
                    '''
                }
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                dir('frontend') {
                    sh '''
                        docker build -t $FRONTEND_IMAGE .
                    '''
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-cred', 
                    usernameVariable: 'DOCKERHUB_USER', 
                    passwordVariable: 'DOCKERHUB_PSW'
                )]) {
                    sh '''
                        echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USER --password-stdin
                        docker push $BACKEND_IMAGE
                        docker push $FRONTEND_IMAGE
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to App Server') {
            steps {
                echo "Deployment steps go here"
                // Example:
                // sh 'ssh user@yourserver "docker pull $BACKEND_IMAGE && docker-compose up -d"'
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
