pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = 'c37034ad-7549-4115-8bc5-8b42f3ac3a5c'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')  
    }
    stages {
        // Stage 1
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
                      image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                      reuseNode true
                  }
              }
              steps{
                sh '''
                  npm install serve
                  node_modules/.bin/serve -s build &
                  sleep 10
                  // Play a test locally
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
        stage('Deploy') {
          agent{
              docker {
                  image 'node:18-alpine'
                  reuseNode true
              }
          }
            steps {
                sh '''
                  npm install netlify-cli
                  node_modules/.bin/netlify --version
                  echo "Deploying to Netlify site ID: $NETLIFY_SITE_ID"
                  node_modules/.bin/netlify status
                  node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('Prod E2E') {
              agent{
                  docker {
                      image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                      reuseNode true
                  }
              }
              environment {
                CI_ENVIRONMENT_URL = 'https://https://soft-beijinho-351299.netlify.app'
              }
              steps {
                sh '''
                  npx playwright test --reporter=html
                '''
              }
              post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
              }
            }
    }
}