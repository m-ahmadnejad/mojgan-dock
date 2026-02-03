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
                  agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.54.2-jammy'
              reuseNode true
            }
                  }
                  environment{
                    E2E_BASE_URL='https://spanish-cards.netlify.app/'
                  }
            
      when { expression { false } }
      steps {
        echo 'E2E disabled (Integration already runs Playwright tests)'
      }
      post{
        always{
         publishHTML([allowMissing:false,alwaysLinkToLastBuild:true,icon:'',keppAll:false,reportDir:'reports-e2e/html/',reportFiles:'index.html',reportName:'Playwright HTML Report',reportTitle:'',userWrapperFileDirectly:true])   
         junit stdioRetention:'ALL',testResult:'reports-e2e/junit.xml'
        }
      }
    }
  }
}
