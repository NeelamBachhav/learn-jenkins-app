pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'YOUR_NETLIFY_SITE_ID'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:18'
                    reuseNode true
                }
            }

            steps {
                bat '''
                    dir
                    node --version
                    npm --version
                    call npm ci
                    call npm run build
                    dir
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18'
                            reuseNode true
                        }
                    }

                    steps {
                        bat '''
                            call npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        bat '''
                            start /B npx serve -s build
                            timeout /t 10 /nobreak > nul
                            call npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Local E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }

        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps {
                bat '''
                    netlify --version

                    echo Deploying to staging. Site ID: %NETLIFY_SITE_ID%

                    call netlify status

                    call netlify deploy --dir=build --json > deploy-output.json

                    for /f "delims=" %%i in ('jq -r ".deploy_url" deploy-output.json') do (
                        set CI_ENVIRONMENT_URL=%%i
                    )

                    echo Staging URL: %CI_ENVIRONMENT_URL%

                    call npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Staging E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'YOUR_NETLIFY_SITE_URL'
            }

            steps {
                bat '''
                    node --version
                    netlify --version

                    echo Deploying to production. Site ID: %NETLIFY_SITE_ID%

                    call netlify status

                    call netlify deploy --dir=build --prod

                    call npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod E2E',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}