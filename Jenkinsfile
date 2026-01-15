pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit') {
            steps {
                bat "python -m pytest test/unit -v --junitxml=unit-test-report.xml"
            }
            post {
                always {
                    junit testResults: 'unit-test-report.xml', allowEmptyResults: true
                }
            }
        }

        stage('Rest') {
            steps {
                script {
                    // Iniciar Wiremock y Flask en segundo plano (Windows)
                    bat 'start /B java -jar wiremock/wiremock.jar --port 9090 --root-dir wiremock'
                    bat 'start /B python -m flask --app app/api run --port 5000'
                    sleep 10
                    bat 'python -m pytest test/rest -v --junitxml=rest-test-report.xml'
                }
            }
            post {
                always {
                    bat 'taskkill /F /IM python.exe 2>nul || echo "No proceso Python para terminar"'
                    bat 'taskkill /F /IM java.exe 2>nul || echo "No proceso Java para terminar"'
                    junit testResults: 'rest-test-report.xml', allowEmptyResults: true
                }
            }
        }

        stage('Static') {
            steps {
                script {
                    // Usar --exit-zero para que flake8 no falle el paso
                    bat 'python -m flake8 . --count --exit-zero > flake8-report.txt'
                    def flake8Output = readFile('flake8-report.txt').trim()
                    echo "Flake8 encontró: ${flake8Output} problemas."
                    
                    // Aplicar umbrales CAMBIANDO EL ESTADO, SIN DETENER EL PIPELINE
                    def numIssues = flake8Output.isInteger() ? flake8Output.toInteger() : 0
                    if (numIssues >= 10) {
                        currentBuild.result = 'FAILURE'
                    } else if (numIssues >= 8) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Security') {
            steps {
                script {
                    // Usar --exit-zero para que bandit no falle el paso
                    bat 'python -m bandit -r . -f json -o bandit-report.json --exit-zero 2>nul'
                    def banditReport = readJSON file: 'bandit-report.json'
                    def totalIssues = banditReport.metrics.total_issues
                    echo "Bandit encontró: ${totalIssues} problemas."
                    
                    // Aplicar umbrales CAMBIANDO EL ESTADO, SIN DETENER EL PIPELINE
                    if (totalIssues >= 4) {
                        currentBuild.result = 'FAILURE'
                    } else if (totalIssues >= 2) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Coverage') {
            steps {
                script {
                    // 1. Ejecutar coverage sobre las pruebas unitarias
                    bat 'python -m coverage run --source=app -m pytest test/unit'
                    // 2. Generar reporte XML (Cobertura) para Jenkins
                    bat 'python -m coverage xml -o coverage.xml'
                    // 3. Generar reporte HTML para inspección manual
                    bat 'python -m coverage html -d coverage_html'
                    
                    // (Opcional) Mostrar un resumen en consola
                    bat 'python -m coverage report'
                }
            }
            post {
                always {
                    // PUBLICAR RESULTADOS - PASO CLAVE
                    // El plugin moderno es 'Coverage', no 'Cobertura' (obsoleto)[citation:1][citation:3].
                    // Si tienes el plugin 'Coverage' instalado[citation:3], usa esta línea:
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']])
                    
                    // **SI FALLA la línea de arriba**, coméntala y prueba con el plugin antiguo (si lo tienes):
                    // publishCoverage adapters: [coberturaAdapter('coverage.xml')]
                }
            }
        }

        stage('Performance') {
            steps {
                script {
                    // 1. Levantar el servicio Flask
                    bat 'start /B python -m flask --app app/api run --port 5000'
                    sleep 5
                    // 2. Ejecutar JMeter en modo no-GUI. Ajusta la ruta a tu JMeter.
                    bat 'C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t jmeter/test-plan.jmx -l jmeter/results.jtl -j jmeter/jmeter.log'
                }
            }
            post {
                always {
                    bat 'taskkill /F /IM python.exe 2>nul || echo "No proceso Python para terminar"'
                    // Publicar resultados de rendimiento
                    perfReport sourceDataFiles: 'jmeter/results.jtl'
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline completado. Resultado final: ${currentBuild.currentResult}"
        }
    }
}