pipeline {
    agent any

    environment {
        DEPLOY_DIR = "deploy"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Deployment') {
            steps {
                bat """
                if not exist ${DEPLOY_DIR} mkdir ${DEPLOY_DIR}
                xcopy /E /I /Y app\\* ${DEPLOY_DIR}\\
                echo "Verifying deployment files..."
                dir ${DEPLOY_DIR}
                """
            }
        }

        stage('Deploy to Render') {
            steps {
                script {
                    echo 'Triggering deployment to Render...'
                    withCredentials([string(credentialsId: 'render-api-token', variable: 'RENDER_API_KEY')]) {
                        def deployUrl = "https://api.render.com/deploy/srv-d007tn7gi27c73b0pbk0?key=" + RENDER_API_KEY
                        def safeDeployUrl = deployUrl.replace(RENDER_API_KEY, '****')
                        echo "Deploy URL: ${safeDeployUrl}"

                        powershell """
                        try {
                            \$response = Invoke-RestMethod -Uri '${deployUrl}' -Method Post -ContentType 'application/json' -Body '{}'
                            Write-Output \$response
                        } catch {
                            Write-Error 'Error in deployment trigger: ' + \$_.Exception.Message
                            exit 1
                        }
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed at ${new Date()}"
            bat "rmdir /S /Q ${DEPLOY_DIR} || exit 0"
        }

        failure {
            echo 'Pipeline execution failed!'
        }
    }
}
