pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '76506490-666f-45c1-9169-b0cef1c06ace'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        // This are the stages of our pipeline, each stage will run in a separate container

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            environment {
                AWS_S3_BUCKET = 'learn-jenkins-29-03-2026'
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        echo "Hello S3!" > index.html
                        aws s3 sync build s3://$AWS_S3_BUCKET
                    '''
                }
            }
        }


//        stage('Tests') {
//            parallel {
//                stage('Unit Tests') {
//                    agent {
//                        docker {
//                            image 'node:18-alpine'
//                            reuseNode true
//                        }
//                    }
//                    steps {
//                        sh '''
//                            #test -f build/index.html
//                            npm test
//                       '''
//                    }
//
//                    post {
//                        always {
//                            junit 'jest-results/junit.xml'
//                        }
//                    }
//                }
//
//                stage('E2E Tests') {
//                    agent {
//                        docker {
//                            image 'my-playwright'
//                            reuseNode true
//                        }
//                    }
//
//                    steps {
//                        sh '''
//                            serve -s build &
//                            sleep 10
//                            npx playwright test --reporter=html
//                        '''
//                    }
//
//                    post {
//                        always {
//                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
//                        }
//                    }
//                }
//            }
//        }
//
//        stage('Deploy Staging') {
//            agent {
//                docker {
//                    image 'my-playwright'
//                    reuseNode true
//                }
//            }
//
//            environment {
//                CI_ENVIRONMENT_URL = 'undefined'
//            }
//
//            steps {
//                sh '''
//                    netlify --version
//                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
//                    netlify status
//                    netlify deploy --dir=build --json > deploy-output.json
//                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
//                    node-jq -r '.deploy_url' deploy-output.json
//                    npx playwright test --reporter=html
//                '''
//            }
//
//            post {
//                always {
//                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
//                }
//            }
//        }
//
//        stage('Deploy prod') {
//            agent {
//                docker {
//                    image 'my-playwright'
//                    reuseNode true
//                }
//            }
//
//            environment {
//                CI_ENVIRONMENT_URL = 'https://marvelous-peony-f5ad92.netlify.app'
//            }
//
//            steps {
//                sh '''
//                    node --version
//                    netlify --version
//                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
//                    netlify status
//                    netlify deploy --dir=build  --prod
//                    npx playwright test --reporter=html
//                '''
//            }
//
//            post {
//                always {
//                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
//                }
//            }
//        }

    }
}