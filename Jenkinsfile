pipeline {
    agent any

    stages {
        // ETAPA 1: Obtener el código
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ETAPA 2: Pruebas Unitarias (¡SOLO SE EJECUTAN UNA VEZ!)
        stage('Unit') {
            steps {
                bat "C:\\Python313\\python.exe -m pytest test\\unit -v --junitxml=unit-test-report.xml"
            }
            post {
                always {
                    junit testResults: 'unit-test-report.xml', allowEmptyResults: true
                }
            }
        }

        // ETAPA 3: Pruebas de Integración REST
        stage('Rest') {
            steps {
                script {
                    echo '--- Iniciando servicios de fondo (Wiremock y Flask) ---'
                    bat 'start /B java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock'
                    bat 'start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
                    sleep 10 // Esperar a que los servicios arranquen
                    
                    echo '--- Ejecutando pruebas REST ---'
                    bat "C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml"
                }
            }
            post {
                always {
                    bat 'taskkill /F /IM python.exe 2>nul || echo "Flask cerrado"'
                    bat 'taskkill /F /IM java.exe 2>nul || echo "Wiremock cerrado"'
                    junit testResults: 'rest-test-report.xml', allowEmptyResults: true
                }
            }
        }

        // ETAPA 4: Análisis de Código Estático (Flake8)
        stage('Static') {
            steps {
                script {
                    bat "C:\\Python313\\python.exe -m flake8 . --count --exit-zero > flake8-report.txt"
                    def report = readFile('flake8-report.txt').trim()
                    echo "Flake8 encontró ${report} problemas."
                    
                    // Aplicar umbrales de la guía
                    def issues = report.isInteger() ? report.toInteger() : 0
                    if (issues >= 10) {
                        currentBuild.result = 'FAILURE'
                        error("Flake8: 10+ hallazgos - BUILD FALLIDO")
                    } else if (issues >= 8) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Flake8: 8-9 hallazgos - BUILD INESTABLE"
                    }
                }
            }
        }

        // ETAPA 5: Pruebas de Seguridad (Bandit)
        stage('Security Test') {
            steps {
                script {
                    bat "C:\\Python313\\python.exe -m bandit -r . -f json -o bandit-report.json 2>nul"
                    def banditReport = readJSON file: 'bandit-report.json'
                    def totalIssues = banditReport.metrics.total_issues
                    echo "Bandit encontró ${totalIssues} problemas de seguridad."
                    
                    // Aplicar umbrales de la guía
                    if (totalIssues >= 4) {
                        currentBuild.result = 'FAILURE'
                        error("Bandit: 4+ hallazgos - BUILD FALLIDO")
                    } else if (totalIssues >= 2) {
                        currentBuild.result = 'UNSTABLE'
                        echo "Bandit: 2-3 hallazgos - BUILD INESTABLE"
                    }
                }
            }
        }

        // ETAPA 6: Pruebas de Cobertura (Coverage)
        stage('Coverage') {
            steps {
                script {
                    // Ejecutar pruebas con cobertura
                    bat "C:\\Python313\\python.exe -m coverage run --source=app -m pytest test\\unit"
                    // Generar reportes
                    bat "C:\\Python313\\python.exe -m coverage xml -o coverage.xml"
                    bat "C:\\Python313\\python.exe -m coverage html -d coverage_html"
                    
                    // Obtener métricas de cobertura
                    def coverageOutput = bat(script: "C:\\Python313\\python.exe -m coverage report --format=total", returnStdout: true).trim()
                    echo "Cobertura total reportada: ${coverageOutput}"
                    
                    // Extraer el porcentaje numérico
                    def coverageLine = coverageOutput.find(/(\d+)%/);
                    def lineCoverage = coverageLine ? coverageLine.replaceAll('[^0-9]', '').toInteger() : 0
                    
                    // Umbrales de la guía para LÍNEAS
                    if (lineCoverage < 85) {
                        currentBuild.result = 'FAILURE'
                    } else if (lineCoverage < 95) {
                        currentBuild.result = 'UNSTABLE'
                    }
                    
                    // OBTENER EL VALOR PARA "LÍNEA 90" (ENTREGABLE ESPECÍFICO)
                    echo "=== INICIO: Búsqueda del valor para 'línea 90' del microservicio suma ==="
                    def appCoverageDetail = bat(script: "C:\\Python313\\python.exe -m coverage report --include=\"app/suma.py\"", returnStdout: true)
                    echo appCoverageDetail
                    echo "=== FIN: Detalle de cobertura para suma.py ==="
                    // Busca manualmente en la salida la línea que diga "90" en la columna de cobertura.
                }
            }
            post {
                always {
                    // Publicar reporte al dashboard de Jenkins (necesita plugin Cobertura)
                    publishCoverage adapters: [coberturaAdapter('coverage.xml')]
                }
            }
        }

        // ETAPA 7: Pruebas de Rendimiento (JMeter)
        stage('Performance') {
            steps {
                script {
                    echo '--- Levantando Flask para las pruebas JMeter ---'
                    bat 'start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
                    sleep 5
                    
                    echo '--- Ejecutando plan de pruebas JMeter ---'
                    // Asegúrate de que la ruta a JMeter y tu test-plan.jmx sean correctas
                    bat 'C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t jmeter\\test-plan.jmx -l jmeter\\results.jtl -j jmeter\\jmeter.log'
                }
            }
            post {
                always {
                    bat 'taskkill /F /IM python.exe 2>nul || echo "Flask cerrado"'
                    // Publicar resultados (necesita plugin Performance)
                    perfReport sourceDataFiles: 'jmeter/results.jtl'
                }
            }
        }
    }

    post {
        always {
            echo "=== PIPELINE FINALIZADO ==="
            echo "Estado final del build: ${currentBuild.currentResult}"
        }
    }
}