pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'master' ? 'nodemain' : 'nodedev'}"
        DOCKER_TAG = 'v1.0'
        HOST_PORT = "${env.BRANCH_NAME == 'master' ? '3000' : '3001'}"
        CONTAINER_NAME = "${env.BRANCH_NAME == 'master' ? 'main-app' : 'dev-app'}"
        DEPLOYMENT_PIPELINE_NAME = "${env.BRANCH_NAME == 'master' ? 'Deploy_to_main' : 'Deploy_to_dev'}"
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

        stage('Push image into docker hub') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-pat', 
                        usernameVariable: 'DOCKER_USER', 
                        passwordVariable: 'DOCKER_PASS'
                        )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
                }
            }
        }

        stage('Trigger deployment pipeline') {
            steps {
                build job: ${env.DEPLOYMENT_PIPELINE_NAME}
            }
        }
    }
}