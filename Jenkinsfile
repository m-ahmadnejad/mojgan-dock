pipeline {
    agent any

    stages {

        /* =========================
           BUILD
        ========================= */
        stage('Build') {
            steps {
                echo 'Installing dependencies'
                sh 'npm ci'
            }
        }

        /* =========================
           TESTS (PARALLEL)
        ========================= */
        stage('Test') {
            parallel {

                /* ---------- UNIT TEST ---------- */
                stage('Unit Tests') {
                    steps {
                        echo 'Running unit tests'
                        sh 'npm test'
                    }
                }

                /* ---------- INTEGRATION TEST (Playwright - Local) ---------- */
                stage('Integration Tests') {
                    agent {
                        docker {
                            // ⚠️ ورژن باید با playwright داخل package.json یکی باشه
                            image 'mcr.microsoft.com/playwright:v1.54.2'
                            reuseNode true
                        }
                    }
                    steps {
                        echo 'Running Playwright integration tests (localhost)'
                        sh 'npx playwright test'
                    }
                }
            }
        }

        /* =========================
           DEPLOY (Mock / Example)
        ========================= */
        stage('Deploy') {
            steps {
                echo 'Deploying application (mock stage)'
                // اینجا معمولاً deploy واقعی انجام میشه
            }
        }

        /* =========================
           END TO END TEST
        ========================= */
        stage('End To End Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.54.2'
                    reuseNode true
                }
            }
            environment {
                // URL واقعی اپلیکیشن
                E2E_BASE_URL = 'https://my-app-dev.example.com'
            }
            steps {
                echo 'Running Playwright end-to-end tests'
                sh 'npx playwright test'
            }
        }
    }
}
