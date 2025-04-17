pipeline {
    agent any

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/your-username/ci-cd-js-project.git'
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ci-cd-js-app .'
            }
        }

        stage('Deploy to Render') {
            steps {
                sh 'curl -X POST https://api.render.com/deploy/srv-RENDER_SERVICE_ID?key=RENDER_API_KEY'
            }
        }
    }
}
