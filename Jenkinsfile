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

        stage('Build') {
            steps {
                bat '''
                    set REACT_APP_VERSION=1.0.%BUILD_NUMBER%

                    call npm ci

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

            post {
                always {
                    echo Unit tests completed
                }
            }
        }

        stage('E2E Tests') {
            steps {
                bat '''
                    call npx playwright install chromium

                    start "" /B npx serve -s build

                    powershell -Command "Start-Sleep -Seconds 10"

                    call npx playwright test --reporter=html
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