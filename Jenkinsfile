pipeline {
    agent any
    
    environment {
        RENDER_API_KEY = credentials('render-api-key')
        RENDER_SERVICE_ID = 'your-render-service-id'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Validation') {
            steps {
                sh 'echo "Validating HTML files..."'
                // Using a basic file existence check
                sh 'find . -name "*.html" | xargs ls -la'
                
                // Optional: Install and use html-validator if needed
                // sh 'npm install -g html-validator-cli'
                // sh 'find . -name "*.html" -exec html-validator --file {} \\;'
            }
        }
        
        stage('Deploy to Render') {
            steps {
                script {
                    echo "Triggering deployment to Render..."
                    
                    def response = httpRequest(
                        url: "https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys",
                        httpMode: 'POST',
                        contentType: 'APPLICATION_JSON',
                        customHeaders: [[name: 'Authorization', value: "Bearer ${RENDER_API_KEY}"]],
                        validResponseCodes: '200:299'
                    )
                    
                    def deployId = readJSON(text: response.content).id
                    echo "Deployment triggered with ID: ${deployId}"
                    
                    // Poll deployment status
                    def deployStatus = "created"
                    def maxAttempts = 30
                    def attempts = 0
                    
                    while (deployStatus != "live" && attempts < maxAttempts) {
                        sleep(10)
                        attempts++
                        
                        def statusResponse = httpRequest(
                            url: "https://api.render.com/v1/services/${RENDER_SERVICE_ID}/deploys/${deployId}",
                            httpMode: 'GET',
                            contentType: 'APPLICATION_JSON',
                            customHeaders: [[name: 'Authorization', value: "Bearer ${RENDER_API_KEY}"]],
                            validResponseCodes: '200:299'
                        )
                        
                        deployStatus = readJSON(text: statusResponse.content).status
                        echo "Deployment status: ${deployStatus} (attempt ${attempts}/${maxAttempts})"
                    }
                    
                    if (deployStatus == "live") {
                        echo "Deployment successful!"
                    } else {
                        error "Deployment failed or timed out"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline execution failed!"
        }
        always {
            echo "Pipeline completed at ${new Date()}"
        }
    }
}