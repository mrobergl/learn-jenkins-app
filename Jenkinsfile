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
        stage('Test') {
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
              Jenkins_Jobs_Pipelines_CICD_and_DevOps_for_Beginners\learn-jenkins-app\node_modules\.bin\serve -s build
              npx playwright test
            '''
          }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}