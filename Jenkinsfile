pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
    }

    stages {

        stage('Environment Check') {
            steps {
                bat '''
                    echo Checking environment...
                    node --version
                    npm --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                bat '''
                    call npm ci
                '''
            }
        }

        stage('Build') {
            steps {
                bat '''
                    set REACT_APP_VERSION=1.0.%BUILD_NUMBER%
                    call npm run build
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                bat '''
                    call npm test -- --watchAll=false
                '''
            }
        }

        stage('Install Playwright Browsers') {
            steps {
                bat '''
                    call npx playwright install chromium
                '''
            }
        }

        stage('E2E Tests') {
            steps {
                bat '''
                    start "" /B npx serve -s build

                    powershell -Command "Start-Sleep -Seconds 10"

                    call npx playwright test
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }

        success {
            echo 'Build successful'
        }

        failure {
            echo 'Build failed'
        }
    }
}