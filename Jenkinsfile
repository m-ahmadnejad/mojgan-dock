pipeline {
  agent any

  options { ansiColor('xterm') }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('E2E Playwright') {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.41.2-jammy'
          reuseNode true
        }
      }
      steps {
        sh 'node -v'
        sh 'npm ci'
        sh 'npx playwright test'
      }
      post {
        always {
          archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
        }
      }
    }
  }
}
