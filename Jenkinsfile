pipeline {
    agent any

    parameters {
        string(name: 'APP_VERSION', description: 'Version to promote to QA (e.g. v1.0.5)')
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
    }

    stages {

        stage('Validate Version') {
            steps {
                script {
                    if (!params.APP_VERSION) {
                        error "APP_VERSION is required!"
                    }

                    echo "Promoting Version: ${params.APP_VERSION}"
                }
            }
        }

        stage('Verify Image Exists (Optional but Recommended)') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh """
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                            docker pull ${DOCKER_IMAGE}:${params.APP_VERSION}
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Trigger Deployment Repo - QA') {
            steps {
                script {
                    build job: 'deployment-repo/qa',
                    parameters: [
                        string(name: 'APP_VERSION', value: "${params.APP_VERSION}")
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
