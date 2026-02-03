pipeline {
  agent any

  options { ansiColor('xterm') }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      agent {
        docker { image 'node:20-alpine' }
      }
      steps {
        sh 'node -v'
        sh 'npm -v'
        sh 'npm ci'
      }
    }

    stage('Unit Tests') {
      agent {
        docker { image 'node:20-alpine' }
      }
      steps {
        sh 'npm ci'
        sh 'npm run test:unit'
      }

      stage('integration tests'){
        agent{
            docker{
                image 'mrc.microsoft.com/playwright:v1.54.2-jammy'
                reuseNode true
            }
        }
      }
    }

    stage('E2E Playwright') {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci'
        sh 'npx playwright test'
      }
      post {
        always {
          archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
          archiveArtifacts artifacts: 'test-results/**', allowEmptyArchive: true
        }
      }
    }
  }
}
