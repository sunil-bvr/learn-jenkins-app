pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = "learnjenkinsapp"
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_DOCKER_REGISTRY= "864981748566.dkr.ecr.us-east-1.amazonaws.com"
        AWS_ECS_CLUSTER = "LearnjenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE = "LearnJenkinsApp-Service-Prod"
        AWS_ECS_TD_PROD = "LearnJenkinsApp-TaskDefinition-Prod"
    }

    stages {        

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
                    ls -al
                '''
            }
        } 

        stage('Build Docker image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'myaws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .                    
                        docker ecr get-login-password | docker login --username --password-stdin AWS $AWS_DOCKER_REGISTRY
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
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
                withCredentials([usernamePassword(credentialsId: 'myaws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/register-task-definition.html#examples
                    sh '''
                        aws --version                        
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')                        
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                    '''
                }
            }
        }
    }
    
}
