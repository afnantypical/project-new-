pipeline {
    agent any

    environment {
        DOCKERHUB_BACKEND  = "syedafnan9148/mean-backend"
        DOCKERHUB_FRONTEND = "syedafnan9148/mean-frontend"

        SSH_HOST = "34.204.186.96"
        SSH_USER = "ubuntu"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_BACKEND}:latest ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t ${DOCKERHUB_FRONTEND}:latest ./frontend'
            }
        }

        stage('Push Images to Docker Hub') {
            environment {
                DOCKERHUB = credentials('dockerhub-cred')
            }
            steps {
                sh '''
                  echo "$DOCKERHUB_PSW" | docker login -u "$DOCKERHUB_USR" --password-stdin
                  docker push ${DOCKERHUB_BACKEND}:latest
                  docker push ${DOCKERHUB_FRONTEND}:latest
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {
                sshagent(credentials: ['app-ec2-ssh']) {
                    sh '''
                      ssh -o StrictHostKeyChecking=no ${SSH_USER}@${SSH_HOST} "
                        cd /opt/mean-app/project-new- &&
                        sudo docker compose pull &&
                        sudo docker compose up -d
                      "
                    '''
                }
            }
        }
    }
}
