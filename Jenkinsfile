pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/eternalscythe/cp1-unir'
            }
        }
        stage('Start Wiremock') {
            steps {
                bat 'java -jar wiremock\\wiremock-standalone-3.12.2.jar --port 8081 --root-dir wiremock &'
                bat 'timeout /t 10'
            }
        }
        stage('Unit Tests') {
            steps {
                bat 'python -m pytest test\\unit -v --junitxml=unit-test-report.xml'
            }
            post {
                always {
                    junit 'unit-test-report.xml'
                }
            }
        }
        stage('Integration Tests') {
            steps {
                bat 'python -m pytest test\\rest -v --junitxml=rest-test-report.xml'
            }
            post {
                always {
                    junit 'rest-test-report.xml'
                }
            }
        }
        stage('Stop Wiremock') {
            steps {
                bat 'taskkill /f /im java.exe 2>nul || echo "Wiremock no estaba corriendo"'
            }
        }
    }
}