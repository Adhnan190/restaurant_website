pipeline {
    agent any
    
    environment {
        // Use credentials binding for security
        RENDER_API_KEY = credentials('render-api-token')
        // Your actual Render service ID
        RENDER_SERVICE_ID = 'srv-d007tn7gi27c73b0pbk0'
        // The name of your site in Render
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
                // Create deployment directory structure using Windows commands
                bat '''
                if not exist deploy mkdir deploy
                xcopy /E /I /Y app\\* deploy\\
                '''
                
                // Validate files were copied correctly
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
                    
                    // Using Deploy Hook for your service
                    def deployHookUrl = "https://api.render.com/deploy/${RENDER_SERVICE_ID}?key=${RENDER_API_KEY}"
                    
                    // Use PowerShell for HTTP requests on Windows
                    def response = bat(script: """
                        powershell -command "Invoke-RestMethod -Uri '${deployHookUrl}' -Method Post -ContentType 'application/json' -Body '{}'"
                    """, returnStdout: true).trim()
                    
                    echo "Deployment triggered: ${response}"
                    
                    // Wait for deployment to complete
                    def deploymentStatus = "in_progress"
                    def attempts = 0
                    def maxAttempts = 30
                    
                    while (deploymentStatus != "live" && attempts < maxAttempts) {
                        sleep(time: 20, unit: 'SECONDS')  // Wait 20 seconds between checks
                        attempts++
                        
                        // Check deployment status using PowerShell
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
                        // Clean up the output to extract just the status
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
            // Clean up deployment directory
            bat 'rmdir /S /Q deploy || exit 0'
        }
    }
}
