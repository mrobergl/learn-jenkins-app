pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
        AWS_ECS_CLUSTER = "arn:aws:ecs:us-east-1:554352105641:cluster/LearningJenkinsApp-Cluster-Prod"
        AWS_ECS_SERVICE = "LearnJenkinsApp-Service-Prod"
        AWS_TASK_DEFINITION = "LearnJenkinsApp-TaskDefinition-Prod"
    }
    stages {
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
        stage ('Build Docker Image') {
          agent any
          steps {
            sh 'docker image build -t myjenkinsapp .'
          }
        }
        stage('Deploy to AWS') {
            agent{
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            environment {
                AWS_DEFAULT_REGION = 'us-east-1'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                  sh '''
                    aws --version
                    yum install -y jq
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service LearnJenkinsApp-Service-Prod --task-definition $AWS_TASK_DEFINITION:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                  '''
                } 
            }
        }
    }
}