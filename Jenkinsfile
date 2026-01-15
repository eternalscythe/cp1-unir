pipeline {
    agent any  // <--- CAMBIO AQUÍ: Que lo ejecute quien esté disponible

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Setup & Integration Tests') {
            steps {
                script {
                    echo "--- Iniciando Servidores en Background ---"
                    // NOTA IMPORTANTE: Al usar agent any, asegúrate de que la máquina
                    // donde corre Jenkins tiene Python y Java instalados en estas rutas.
                    
                    // Wiremock
                    bat "start /B java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock"
                    
                    // Flask
                    bat "start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000"
                    
                    echo "--- Esperando a que levanten los servicios (10s) ---"
                    sleep 10
                    
                    echo "--- Ejecutando Tests ---"
                    try {
                        bat "C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml"
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Los tests fallaron pero continuamos: ${e.message}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "--- Limpiando entorno ---"
            bat 'taskkill /F /IM python.exe 2>nul || echo "Flask ya estaba cerrado"'
            junit testResults: 'rest-test-report.xml', allowEmptyResults: true
        }
    }
}