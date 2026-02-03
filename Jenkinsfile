pipeline {
  agent any
  options {
    ansiColor('xterm')
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

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
            sh 'npm -v'
            sh 'npm ci'

            // اگر reporter در playwright.config.ts درست تنظیم باشد:
            // HTML -> playwright-report/
            // JUnit -> test-results/junit.xml
            sh 'npx playwright test'
          }
          post {
            always {
              // 1) HTML Report (لینک مستقیم در Jenkins)
              publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report'
              ])

              // 2) JUnit Report (برای trend و test result در Jenkins)
              junit testResults: 'test-results/junit.xml', allowEmptyResults: true

              // 3) آرشیو فایل‌ها برای دانلود/بررسی
              archiveArtifacts artifacts: 'playwright-report/**', allowEmptyArchive: true
              archiveArtifacts artifacts: 'test-results/**', allowEmptyArchive: true
            }
          }
        }

      }
    }

    // (اختیاری) Stage واقعی E2E بعد از Deploy
    // اگر خواستی بعداً فعالش کنی، وقتی deploy واقعی داشتی:
    /*
    stage('E2E Playwright') {
      agent {
        docker {
          image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
          reuseNode true
        }
      }
      environment {
        E2E_BASE_URL = 'https://spanish-cards.netlify.app/'
      }
      steps {
        sh 'npm ci'
        sh 'npx playwright test'
      }
      post {
        always {
          publishHTML(target: [
            allowMissing: true,
            alwaysLinkToLastBuild: true,
            keepAll: true,
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright HTML Report'
          ])
          junit testResults: 'test-results/junit.xml', allowEmptyResults: true
        }
      }
    }
    */
  }
}
