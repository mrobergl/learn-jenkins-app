pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
        APP_NAME="learnjenkinsapp"
        AWS_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = "arn:aws:ecs:us-east-1:554352105641:cluster/LearningJenkinsApp-Cluster-Prod"
        AWS_DOCKER_REGISTRY = "554352105641.dkr.ecr.us-east-1.amazonaws.com"
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
          agent{
              docker {
                  image 'my-aws-cli'
                  reuseNode true
                  args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
              }
            }
          steps {
            withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
              sh '''
                docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
            }
          }
        }
        stage('Deploy to AWS') {
            agent{
                docker {
                    image 'my-aws-cli'
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
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service LearnJenkinsApp-Service-Prod --task-definition $AWS_TASK_DEFINITION:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE
                  '''
                } 
            }
        }
    }
}