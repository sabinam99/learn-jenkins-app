pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learn-jenkins-app'
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_DOCKER_REGISTRY = '886058822965.dkr.ecr.us-east-1.amazonaws.com'
        AWS_ECS_CLUSTER = 'learn-jenkins'
        AWS_ECS_SERVICE = 'learn-jenkins-service'
        AWS_ECS_TD = 'learn-jenkins-taks-definition'
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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }


            steps {
                sh '''
                    docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password | docker login -username AWS --password-stdin $AWS_DOCKER_REGISTRY
                    docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition.json | jq -r '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }



    }
}