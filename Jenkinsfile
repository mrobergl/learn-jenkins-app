pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
    }
    stages {
          stage('Deploy to AWS') {
            agent{
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_DEFAULT_REGION = 'us-east-1'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                  sh '''
                    aws --version
                    aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                    aws ecs update-service --cluster arn:aws:ecs:us-east-1:554352105641:cluster/LearningJenkinsApp-Cluster-Prod --service LearnJenkinsApp-Service-Prod --task-definition LearnJenkinsApp-TaskDefinition-Prod:2
                  '''
                } 
            }
        }
        stage('Build') {
          agent{
              docker {
                  image 'node:18-alpine'
                  reuseNode true
              }
          }
            steps {
                echo 'Building..'
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
    }
}