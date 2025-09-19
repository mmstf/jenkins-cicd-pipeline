@Library('DeployToMaster') _
pipeline {
    agent {
        label 'docker'
    }

    tools {
        nodejs 'node'
    }

    environment {
        DOCKER_IMAGE = "${env.BRANCH_NAME == 'master' ? 'mmstf/nodemain' : 'mmstf/nodedev'}"
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

        stage('Check Dockerfile with Hadolint') {
            steps {
                script {
                    sh '''
                        hadolint -t error Dockerfile
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Scan Docker Image for vunerabilities') {
            steps {
                script {
                    def dockerImage = "${DOCKER_IMAGE}:${DOCKER_TAG}"
                    def vulnerabilities = sh(script: "trivy image --exit-code 0 --severity HIGH,MEDIUM,LOW --no-progress ${dockerImage}", returnStdout: true).trim()
                    echo "Vulnerability Report:\n${vulnerabilities}"
                }
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
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    '''
                }
                }
            }
        }

        stage('Trigger dev deployment pipeline') {
            when {
                branch 'dev'
            }
            steps {
                build job: "Deploy_to_dev" // "${env.DEPLOYMENT_PIPELINE_NAME}" // hard coded dev deployment
            }
        }

        stage('Trigger master deployment pipeline') {
            when {
                branch 'master'
            }
            steps {
                deployToMaster()
            }
        }
    }
}