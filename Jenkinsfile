pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        DOCKER_IMAGE = "sony9014/mydeploy"
        NEXUS_URL = "http://172.31.42.87:8081"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Create Pull Request to Dev') {
            steps {
                script {

                    // 🔥 Get current branch dynamically
                    def branchName = env.BRANCH_NAME
                    echo "Current Branch: ${branchName}"

                    withCredentials([string(credentialsId: 'github-api-creds', variable: 'GITHUB_TOKEN')]) {

                        // 🔍 Check if PR already exists
                        def prCheck = sh(
                            script: """
                            curl -s -H "Authorization: token $GITHUB_TOKEN" \
                            https://api.github.com/repos/sonyvinny77/application-repo/pulls?head=sonyvinny77:${branchName}&base=dev
                            """,
                            returnStdout: true
                        ).trim()

                        if (prCheck.contains('"number"')) {
                            echo "✅ PR already exists. Skipping creation."
                        } else {
                            echo "🚀 Creating Pull Request..."

                            // 🆕 Create PR
                            def response = sh(
                                script: """
                                curl -s -X POST https://api.github.com/repos/sonyvinny77/application-repo/pulls \
                                -H "Authorization: token $GITHUB_TOKEN" \
                                -H "Accept: application/vnd.github+json" \
                                -d '{
                                  "title": "Auto PR: ${branchName} → dev",
                                  "head": "${branchName}",
                                  "base": "dev",
                                  "body": "Created automatically by Jenkins pipeline."
                                }'
                                """,
                                returnStdout: true
                            ).trim()

                            echo "📌 GitHub API Response:"
                            echo "${response}"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline Completed Successfully"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
