pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                sh 'npm ci'
                sh 'CI=true npm test -- --watchAll=false'
            }
        }
    }
}
