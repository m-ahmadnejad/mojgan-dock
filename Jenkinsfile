pipeline {
  agent any

  options {
    ansiColor('xterm')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install') {
      agent {
        docker {
          image 'node:22-alpine'
        }
      }
      steps {
        sh 'node -v'
        sh 'npm ci'
      }
    }

    stage('Unit tests') {
      agent {
        docker {
          image 'node:22-alpine'
          reuseNode true
        }
      }
      steps {
        // If you have vitest configured in package.json, this will work.
        // If not, Jenkins will fail here and we can adjust.
        sh 'npm test -- --run'
      }
    }

    stage('E2E Playwright') {
      agent {
        docker {
          // Playwright needs browsers; this image includes them.
          image 'mcr.microsoft.com/playwright:v1.41.2-jammy'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci'
        sh 'npx playwright install --with-deps'
        sh 'npx playwright test'
      }
      post {
        always {
          // If Playwright report exists, archive it
          archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
          junit testResults: 'test-results/**/*.xml', allowEmptyResults: true
        }
      }
    }
  }
}
 