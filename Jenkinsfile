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

        stage('Playwright Browser Install') {
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

                    call npx playwright test e2e/app.spec.js --grep "has title|has Jenkins in the body"
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }

        success {
            echo 'SUCCESS: All configured stages passed'
        }

        failure {
            echo 'FAILURE: One or more stages failed'
        }
    }
}
``