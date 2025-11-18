pipeline {
    agent any

    tools {
        jdk "jdk17"
        nodejs "node20"
    }

    environment {
        DOCKER_IMAGE = "krnikhi26/devops-project"
        IMAGE_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "devops-project"
        APP_PORT = "80"
        EMAIL = "krnpsbb@gmail.com"
        SERVER_IP = "13.201.86.242"
    }

    triggers {
        githubPush()   // Automatically triggers build when code is pushed to GitHub
    }

    stages {
        stage('Initialize') {
            steps {
                echo '--- Nikhitha K R | IoT A | 22011102036 ---'
            }
        }

        stage('Checkout Repository') {
            steps {
                echo '========== Cloning Repository =========='
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '========== Installing npm dependencies =========='
                sh '''
                    node --version
                    npm --version
                    npm install
                '''
            }
        }

        stage('Build Application') {
            steps {
                echo '========== Building React application =========='
                sh 'npm run build'
            }
        }

        stage('Run Tests') {
            steps {
                echo '========== Running Tests =========='
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh 'npm test || true'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '========== Building Docker Image =========='
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                echo '========== Pushing Docker Image to DockerHub =========='
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh """
                            docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                            docker push ${DOCKER_IMAGE}:latest
                            echo "Successfully pushed image to DockerHub"
                        """
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                echo '========== Deploying Container =========='
                script {
                    sh """
                        # Stop and remove any old container
                        docker stop ${CONTAINER_NAME} 2>/dev/null || true
                        docker rm ${CONTAINER_NAME} 2>/dev/null || true
                        
                        # Remove dangling images to free space
                        docker image prune -f || true

                        # Pull latest image
                        docker pull ${DOCKER_IMAGE}:latest
                        
                        # Run new container
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            -p ${APP_PORT}:80 \
                            --restart unless-stopped \
                            ${DOCKER_IMAGE}:latest
                        
                        # Wait a few seconds to start
                        sleep 5

                        # Verify container is running
                        if docker ps | grep -q ${CONTAINER_NAME}; then
                            echo "Container ${CONTAINER_NAME} is running successfully"
                            docker ps | grep ${CONTAINER_NAME}
                        else
                            echo "Container failed to start"
                            docker logs ${CONTAINER_NAME}
                            exit 1
                        fi
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                echo '========== Performing Health Check =========='
                script {
                    sh """
                        sleep 10
                        echo "Testing application at http://${SERVER_IP}:${APP_PORT}"
                        if curl -f http://${SERVER_IP}:${APP_PORT} > /dev/null 2>&1; then
                            echo "Health check passed - Application is responding"
                        else
                            echo "Health check failed - Application is not responding"
                            docker logs ${CONTAINER_NAME}
                            exit 1
                        fi
                    """
                }
            }
        }
    }

    post {
        success {
            echo '========== Build and Deployment Successful! =========='
            emailext(
                to: "${EMAIL}",
                subject: "SUCCESS: Jenkins Pipeline Build #${BUILD_NUMBER}",
                body: """
                    <html>
                    <body style="font-family: Arial, sans-serif;">
                        <div style="background-color: #4CAF50; color: white; padding: 20px; text-align: center;">
                            <h2>Build Successful!</h2>
                        </div>
                        <div style="padding: 20px;">
                            <h3>Build Details</h3>
                            <table border="1" cellspacing="0" cellpadding="8">
                                <tr><td><b>Build Number</b></td><td>${BUILD_NUMBER}</td></tr>
                                <tr><td><b>Docker Image</b></td><td>${DOCKER_IMAGE}:${IMAGE_TAG}</td></tr>
                                <tr><td><b>Server</b></td><td>${SERVER_IP}</td></tr>
                                <tr><td><b>Application URL</b></td>
                                    <td><a href="http://${SERVER_IP}" target="_blank">http://${SERVER_IP}</a></td></tr>
                            </table>
                            <p style="margin-top: 20px;">This is an automated notification from Jenkins CI/CD.</p>
                        </div>
                    </body>
                    </html>
                """,
                mimeType: 'text/html'
            )
        }

        failure {
            echo '========== Build Failed! =========='
            emailext(
                to: "${EMAIL}",
                subject: "FAILED: Jenkins Pipeline Build #${BUILD_NUMBER}",
                body: """
                    <html>
                    <body style="font-family: Arial, sans-serif;">
                        <div style="background-color: #f44336; color: white; padding: 20px; text-align: center;">
                            <h2>Build Failed!</h2>
                        </div>
                        <div style="padding: 20px;">
                            <h3>Build Details</h3>
                            <table border="1" cellspacing="0" cellpadding="8">
                                <tr><td><b>Build Number</b></td><td>${BUILD_NUMBER}</td></tr>
                                <tr><td><b>Docker Image</b></td><td>${DOCKER_IMAGE}:${IMAGE_TAG}</td></tr>
                                <tr><td><b>Check Jenkins Logs</b></td><td><a href="${BUILD_URL}console">View Console Log</a></td></tr>
                            </table>
                            <p style="margin-top: 20px;">Please check the Jenkins console output for detailed errors.</p>
                        </div>
                    </body>
                    </html>
                """,
                mimeType: 'text/html'
            )
        }

        always {
            echo '========== Cleaning up workspace =========='
            deleteDir()
        }
    }
}
