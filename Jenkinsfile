pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Get Latest Docker Version') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        env.APP_VERSION = sh(
                            script: """
                                curl -s -u \$DOCKER_USER:\$DOCKER_PASS https://hub.docker.com/v2/repositories/${DOCKER_IMAGE}/tags?page_size=100 | \
                                jq -r '.results[].name' | sort -V | tail -n1
                            """,
                            returnStdout: true
                        ).trim()
                    }

                    echo "Deploying to PROD Version: ${env.APP_VERSION}"
                }
            }
        }

        stage('Trigger PROD Deployment') {
            steps {
                script {
                    build job: 'deployment-repo/prod',
                    parameters: [
                        string(name: 'APP_VERSION', value: "${env.APP_VERSION}")
                    ]
                }
            }
        }
    }
}
