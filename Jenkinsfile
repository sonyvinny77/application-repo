pipeline {
    agent any

    environment {
        APP_NAME = "myapp"
        DOCKER_REPO = "sony9014/myapp"
        VERSION = "v1.0.30"
        PROD_SERVER = "ec2-user@172.31.5.86"
    }

    stages {

        stage('Manual Approval') {
            steps {
                input message: "Approve deployment to PRODUCTION?"
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate Version') {
            steps {
                echo "Deploying Version: ${VERSION}"
            }
        }

        stage('Pull Docker Image') {
            steps {
                sh "docker pull ${DOCKER_REPO}:${VERSION}"
            }
        }

        stage('Deploy to Production') {
            steps {
                sshagent(['docker-server-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${PROD_SERVER} '
                    docker stop ${APP_NAME} || true
                    docker rm ${APP_NAME} || true
                    docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_REPO}:${VERSION}
                    '
                    """
                }
            }
        }

        stage('Success') {
            steps {
                echo "Production Deployment Completed 🚀"
            }
        }
    }

    post {
        success {
            echo "PROD Deployment Successful ✅"
        }
        failure {
            echo "PROD Deployment Failed ❌"
        }
    }
}
