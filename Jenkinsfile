pipeline {
    agent any

    environment {
        RENDER_SERVICE_ID = 'srv-d007tn7gi27c73b0pbk0'
        RENDER_SITE_NAME = 'tastebuds-restaurant'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Deployment') {
            steps {
                bat '''
                if not exist deploy mkdir deploy
                xcopy /E /I /Y app\\* deploy\\
                '''
                bat '''
                echo "Verifying deployment files..."
                dir deploy
                '''
            }
        }

        stage('Deploy to Render') {
            steps {
                script {
                    echo "Triggering deployment to Render..."
                    withCredentials([string(credentialsId: 'render-api-token', variable: 'RENDER_API_KEY')]) {
                        def deployHookUrl = "https://api.render.com/deploy/${RENDER_SERVICE_ID}?key=${RENDER_API_KEY}"

                        def response = bat(script: """
                            powershell -command "Invoke-RestMethod -Uri '${deployHookUrl}' -Method Post -ContentType 'application/json' -Body '{}'"
                        """, returnStdout: true).trim()

                        echo "Deployment triggered: ${response}"

                        // Check deployment status
                        def deploymentStatus = "in_progress"
                        def attempts = 0
                        def maxAttempts = 30

                        while (deploymentStatus != "live" && attempts < maxAttempts) {
                            sleep(time: 20, unit: 'SECONDS')
                            attempts++

                            def statusCommand = """
                                powershell -command "
                                    \$headers = @{
                                        Authorization = 'Bearer ${RENDER_API_KEY}'
                                    }
                                    \$response = Invoke-RestMethod -Uri 'https://api.render.com/v1/services/${RENDER_SERVICE_ID}' -Headers \$headers -Method Get
                                    Write-Output \$response.status
                                "
                            """

                            deploymentStatus = bat(script: statusCommand, returnStdout: true).trim()
                            deploymentStatus = deploymentStatus.replaceAll(".*\\r\\n", "")

                            echo "Current deployment status: ${deploymentStatus} (attempt ${attempts}/${maxAttempts})"
                        }

                        if (deploymentStatus == "live") {
                            echo "Deployment successful! Site is live at: https://${RENDER_SITE_NAME}.onrender.com"
                        } else {
                            error "Deployment failed or timed out after ${attempts} attempts"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
            echo "Your TasteBuds restaurant website is now deployed!"
        }
        failure {
            echo "Pipeline execution failed!"
        }
        always {
            echo "Pipeline completed at ${new Date()}"
            bat 'rmdir /S /Q deploy || exit 0'
        }
    }
}
