pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {               

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
        

            steps {
                withCredentials([usernamePassword(credentialsId: 'myaws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/register-task-definition.html#examples
                    sh '''
                        aws --version                    
                        aws s3 ls
                        # aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                    '''
                }
            }                     
        }

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
    }
    
}
