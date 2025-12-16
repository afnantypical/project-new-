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
                        docker push ${DOCKERHUB_BACKEND}:latest
                        docker push ${DOCKERHUB_FRONTEND}:latest
                    '''
                }
            }
        }

        stage('Deploy to EC2 Server') {
            steps {
                sshagent(['app-ec2-ssh']) {
                    sh
