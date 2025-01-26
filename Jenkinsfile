pipeline {
    agent any
    stages {
        // Stage 1
        /* stage('Build') {
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
        } */
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
                  npx playwright test
                  npx playwright test --reporter=html
                '''
              }
              post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrite HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                }
              }
            }
          }
        }
        
    }
   
}