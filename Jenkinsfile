pipeline {
    // Usamos el nombre que aparece en tu log. 
    // Asegúrate en "Administrar Jenkins > Nodes" que este es el nombre correcto.
    agent { label 'agente-windows-2' } 

    stages {
        stage('Checkout') {
            steps {
                // Descarga el código (Jenkins lo suele hacer auto, pero por si acaso)
                checkout scm
            }
        }
        
        stage('Setup & Integration Tests') {
            steps {
                script {
                    echo "--- Iniciando Servidores en Background ---"
                    // 'start /B' es CRUCIAL en Windows para que no bloquee la ejecución
                    // Lanzamos Wiremock
                    bat "start /B java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock"
                    
                    // Lanzamos Flask
                    // Ajusta la ruta de Python si es diferente en tu agente
                    bat "start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000"
                    
                    echo "--- Esperando a que levanten los servicios (10s) ---"
                    sleep 10
                    
                    echo "--- Ejecutando Tests ---"
                    // Ejecutamos pytest
                    // Si falla el test, queremos que marque la build como inestable/fallida pero que llegue al post
                    try {
                        bat "C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml"
                    } catch (Exception e) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Los tests fallaron: ${e.message}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "--- Limpiando entorno ---"
            
            // 1. Matamos Python (Flask). Esto es seguro si Jenkins no corre sobre Python.
            bat 'taskkill /F /IM python.exe 2>nul || echo "Flask ya estaba cerrado"'
            
            // 2. PARA WIREMOCK (JAVA):
            // NO usamos taskkill /IM java.exe porque mataríamos a Jenkins.
            // Opción segura: Intentamos cerrarlo vía request HTTP (Wiremock admin API)
            // Si no tienes curl en Windows, puedes ignorar este paso, o intentar filtrar mejor el taskkill.
            // Por ahora, como es un entorno de lab, lo más seguro es NO matar Java automáticamente.
            // Si necesitas cerrarlo, busca el proceso a mano o usa:
            // bat 'wmic process where "CommandLine like \'%wiremock%\'" call terminate'
            
            // Publicamos resultados si existen
            junit testResults: 'rest-test-report.xml', allowEmptyResults: true
        }
    }
}