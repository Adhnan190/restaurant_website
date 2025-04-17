pipeline {
    agent any
    
    environment {
        RENDER_API_KEY = credentials('render-api-token')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Deployment') {
            steps {
                script {
                    // Create deploy directory and copy the necessary files
                    bat 'if not exist deploy mkdir deploy'
                    bat 'xcopy /E /I /Y app\\* deploy\\'
                    echo "Verifying deployment files..."
                    bat 'dir deploy'
                }
            }
        }

        stage('Deploy to Render') {
            steps {
                withCredentials([string(credentialsId: 'render-api-token', variable: 'RENDER_API_KEY')]) {
                    powershell """
                    try {
                        \$response = Invoke-RestMethod -Uri "https://api.render.com/deploy/srv-d007tn7gi27c73b0pbk0?key=\$env:RENDER_API_KEY" -Method Post -ContentType 'application/json' -Body '{}'
                        Write-Output \$response
                    } catch {
                        Write-Error ("Error in deployment trigger: \$($_.Exception.Message)")
                        exit 1
                    }
                    """
                }
            }
        }

        stage('Post Actions') {
            steps {
                echo "Pipeline completed"
                bat 'rmdir /S /Q deploy || exit 0'
            }
        }
    }
}
