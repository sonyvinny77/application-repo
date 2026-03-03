pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    environment {
        DOCKER_IMAGE = "sony9014/myapp"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Initialization - Auto Version Increment') {
    steps {
        script {

            sh "git fetch --tags"

            // Get latest tag or default
            def latestTag = sh(
                script: "git describe --tags --abbrev=0 || echo v1.0.0",
                returnStdout: true
            ).trim()

            echo "Latest Tag: ${latestTag}"

            // Remove 'v'
            def version = latestTag.replace("v", "")
            def parts = version.tokenize(".")

            def major = parts[0].toInteger()
            def minor = parts[1].toInteger()
            def patch = parts[2].toInteger()

            // Increment patch
            patch = patch + 1

            env.APP_VERSION = "v${major}.${minor}.${patch}"

            echo "New Version: ${APP_VERSION}"

            // Create new tag
            sh """
               git tag ${APP_VERSION}
               git push origin ${APP_VERSION}
            """
        }
    }
}

        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker build --no-cache -t ${DOCKER_IMAGE}:${APP_VERSION} .
                        docker push ${DOCKER_IMAGE}:${APP_VERSION}
                        docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy to Dev Server') {
            steps {
                script {
                    sshagent(credentials: ['docker-server-ssh']) {
                        sh """
                        ssh -o StrictHostKeyChecking=no ec2-user@18.216.228.137 "
                            docker pull ${DOCKER_IMAGE}:${APP_VERSION} &&
                            docker stop app || true &&
                            docker rm app || true &&
                            docker run -d -p 8080:8080 --restart=always --name app ${DOCKER_IMAGE}:${APP_VERSION}
                        "
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ DEV Deployment Successful!"
        }
        failure {
            echo "❌ DEV Deployment Failed!"
        }
    }
}
