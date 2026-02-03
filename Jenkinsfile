pipeline {
  agent any
  options { ansiColor('xterm') }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    // Build فقط برای اینکه مطمئن شیم npm/lockfile درست هستند
    stage('Build') {
      agent { docker { image 'node:20-alpine' } }
      steps {
        sh 'node -v'
        sh 'npm -v'
        sh 'npm ci'
      }
    }

    // ✅ Parallel execution
    stage('Tests (Parallel)') {
      parallel {
        stage('Unit Tests') {
          agent { docker { image 'node:20-alpine' } }
          steps {
            sh 'node -v'
            sh 'npm -v'
            sh 'npm ci'
            sh 'npm run test:unit'
          }
        }

        stage('Integration Tests (Playwright)') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
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
              archiveArtifacts artifacts: 'test-results/**', allowEmptyArchive: true
            }
          }
        }
      }
    }

    // ❗ فعلاً غیر فعال تا تست‌ها دوبار اجرا نشن
    stage('E2E Playwright') {
      when { expression { false } }
      steps {
        echo 'E2E disabled (Integration already runs Playwright tests)'
      }
    }
  }
}
