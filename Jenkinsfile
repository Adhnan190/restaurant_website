pipeline {
    agent any

    environment {
        RENDER_API_KEY = credentials('render-api-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Adhnan190/restaurant_website.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                bat 'npm test'
            }
        }

        stage('Prepare Deployment') {
            steps {
                bat '''
                    if not exist deploy mkdir deploy
                    xcopy /E /I /Y app\\* deploy\\
                    echo "Verifying deployment files..."
                    dir deploy
                '''
            }
        }

        stage('Deploy to Render') {
            steps {
                echo 'Triggering deployment to Render...'
                script {
                    withCredentials([string(credentialsId: 'render-api-token', variable: 'RENDER_API_KEY')]) {
                        bat """
                            powershell -Command "Invoke-RestMethod -Uri 'https://api.render.com/deploy/srv-d007tn7gi27c73b0pbk0?key=$RENDER_API_KEY'"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed at ${new Date()}"
            bat 'rmdir /S /Q deploy || exit 0'
        }
        failure {
            echo "Pipeline execution failed!"
        }
    }
}
