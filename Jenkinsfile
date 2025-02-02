pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'c37034ad-7549-4115-8bc5-8b42f3ac3a5c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_NUMBER}"
    }
    stages {
        stage('AWS') {
            agent{
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                  sh '''
                    aws --version
                    echo "I love Elaheh" > index.html
                    aws s3 cp index.html s3://learn-jenkins-02022025/index.html
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
        stage ('Run Tests'){
          parallel {
            stage('Unit test') {
              agent{
                  docker {
                      image 'node:18-alpine'
                      reuseNode true
                  }
              }
              steps{
                sh '''
                  test -f build/index.html
                  npm test
                '''
              }
              post {
                always {
                    junit 'jest-results/junit.xml'
                }
              }
            }
            stage('E2E') {
              agent{
                  docker {
                      image 'my-playwright'
                      reuseNode true
                  }
              }
              steps{
                sh '''
                  serve -s build &
                  sleep 10
                  npx playwright test --reporter=html
                '''
              }
              post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite local', reportTitles: '', useWrapperFileDirectly: true])
                }
              }
            }
          }
        }
        stage('Deploy staging') {
              agent{
                  docker {
                      image 'my-playwright'
                      reuseNode true
                  }
              }
              environment {
                CI_ENVIRONMENT_URL = "STAGING_RUL_TO_BE_SET"
              }
              steps {
                sh '''
                  netlify --version
                  echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                  netlify status
                  netlify deploy --dir=build --json > deploy-output.json
                  CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
                  npx playwright test --reporter=html
                '''
              }
              post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
              }
        }
        stage('Approval for Prod') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure, I want to deploy!'
                }
            }
        }
        stage('Deploy prod') {
              agent{
                  docker {
                      image 'my-playwright'
                      reuseNode true
                  }
              }
              environment {
                CI_ENVIRONMENT_URL = 'http://soft-beijinho-351299.netlify.app'
              }
              steps {
                sh '''
                  node --version
                  netlify --version
                  echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                  netlify status
                  deploy --dir=build --prod
                  npx playwright test --reporter=html
                '''
              }
              post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
              }
            }
    }
}