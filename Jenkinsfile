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

        stage('Determine Latest Docker Version') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        // Login and get latest tag from Docker
                        env.APP_VERSION = sh(
                            script: """
                                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                                # List all tags, sort, pick latest (lexicographic order works if your tags are semantic)
                                docker pull ${DOCKER_IMAGE}:latest || true
                                docker logout
                                
                                # Optional: if you push semantic tags, get the latest tag via API
                                curl -s -u \$DOCKER_USER:\$DOCKER_PASS https://hub.docker.com/v2/repositories/${DOCKER_IMAGE}/tags?page_size=100 | \
                                jq -r '.results[].name' | sort -V | tail -n1
                            """,
                            returnStdout: true
                        ).trim()
                    }

                    echo "Promoting Docker Version: ${env.APP_VERSION}"

                    if (!env.APP_VERSION) {
                        error "No Docker version found!"
                    }
                }
            }
        }

        stage('Trigger Deployment Repo - QA') {
            steps {
                script {
                    build job: 'deployment-repo/qa',
                    parameters: [
                        string(name: 'APP_VERSION', value: "${env.APP_VERSION}")
                    ]
                }
            }
        }
    }

    post {
        success {
            echo "✅ Release Pipeline Completed - QA Triggered"
        }
        failure {
            echo "❌ Release Pipeline Failed"
        }
    }
}
