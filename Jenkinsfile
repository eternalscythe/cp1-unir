pipeline {
    agent none // Definimos el agente en cada etapa

    stages {
        // Etapa 1: Iniciar Wiremock (se ejecutará en primer plano, bloqueando este paso)
        stage('Start Wiremock') {
            agent { label 'agente2' } // Mismo agente que las pruebas
            steps {
                bat 'java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock'
            }
        }
        // Etapa 2 (en paralelo): Iniciar Flask y ejecutar pruebas
        stage('Run Integration Tests') {
            parallel {
                stage('Start Flask App') {
                    steps {
                        bat 'C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
                    }
                }
                stage('Execute Tests') {
                    steps {
                        // Esperar un momento a que Flask arranque
                        bat 'ping -n 10 127.0.0.1 >nul'
                        // Ejecutar las pruebas de integración
                        bat 'C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml'
                    }
                }
            }
        }
    }
    post {
        always {
            // Limpieza garantizada: detener los servicios
            bat 'taskkill /F /IM java.exe 2>nul || echo "No Wiremock process"'
            bat 'taskkill /F /IM python.exe 2>nul || echo "No Flask process"'
        }
    }
}