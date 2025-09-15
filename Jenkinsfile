pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'master' ? 'nodemain' : 'nodedev'}"
        DOCKER_TAG = 'v1.0'
        HOST_PORT = "${env.BRANCH_NAME == 'master' ? '3000' : '3001'}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh "docker run -d --expose 3000 -p ${HOST_PORT}:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }

    port {
        always {
            cleanWs()
        }
    }
}