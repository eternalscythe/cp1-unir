pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/eternalscythe/cp1-unir.git'
            }
        }
        stage('Start Wiremock') {
            steps {
                bat 'start /B java -jar wiremock\\wiremock.jar --port 8081 --root-dir wiremock'
                bat 'ping -n 11 127.0.0.1 >nul'
            }
        }
        stage('Unit Tests') {
            steps {
                bat 'C:\\Python313\\python.exe -m pytest test\\unit -v --junitxml=unit-test-report.xml'
            }
            post {
                always {
                    junit 'unit-test-report.xml'
                }
            }
        }
        stage('Integration Tests') {
            steps {
                bat 'C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml'
            }
            post {
                always {
                    junit 'rest-test-report.xml'
                }
            }
        }
        stage('Stop Wiremock') {
            steps {
                bat 'taskkill /f /im java.exe 2>nul || echo "Wiremock ya detenido"'
            }
        }
    }
}