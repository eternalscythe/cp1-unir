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
        // 1. INICIAR WIREMOCK EN EL PUERTO CORRECTO (9090, no 8081)
        bat 'start /B java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock'
        // Espera 5 segundos (usa ping para simular espera)
        bat 'ping -n 6 127.0.0.1 >nul'
    }
}
stage('Start Flask App') {
    steps {
        // 2. INICIAR LA APLICACIÓN FLASK EN SEGUNDO PLANO
        bat 'start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
        // Espera a que Flask esté listo (5 segundos)
        bat 'ping -n 6 127.0.0.1 >nul'
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