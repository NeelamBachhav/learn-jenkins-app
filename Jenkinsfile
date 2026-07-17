pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
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
                    dir
                    call npm ci
                    call npm run build
                    dir
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
                    junit allowEmptyResults: true, testResults: 'jest-results/*.xml'
                }
            }
        }

        stage('E2E Tests') {
            steps {
                bat '''
                    call npm run e2e -- --headless
                    

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