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
                    sh """
                        OLD_CONTAINER=\$(docker ps -q --filter name=${CONTAINER_NAME})
                        if [ "\$OLD_CONTAINER" ]; then
                            docker stop \$OLD_CONTAINER
                            docker rm \$OLD_CONTAINER
                        fi
                    """

                    sh "docker run -d --name ${CONTAINER_NAME} -p ${HOST_PORT}:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}"
                }
            }
        }
    }
}