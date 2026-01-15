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
                // RUTA ABSOLUTA AL EJECUTABLE DE PYTHON
                bat "C:\\Python313\\python.exe -m pytest test\\unit -v --junitxml=unit-test-report.xml"
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
                    // Iniciar servicios con la ruta absoluta
                    bat 'start /B java -jar wiremock\\wiremock.jar --port 9090 --root-dir wiremock'
                    bat 'start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
                    sleep 10
                    bat 'C:\\Python313\\python.exe -m pytest test\\rest -v --junitxml=rest-test-report.xml'
                }
            }
            post {
                always {
                    // Matar solo procesos Python para no cerrar Jenkins
                    bat 'taskkill /F /IM python.exe 2>nul || echo "No proceso Python para terminar"'
                    // OPCIÓN SEGURA: NO MATAR JAVA. Wiremock se cerrará al final del workspace.
                    // bat 'taskkill /F /IM java.exe 2>nul || echo "No proceso Java para terminar"'
                    junit testResults: 'rest-test-report.xml', allowEmptyResults: true
                }
            }
        }

        stage('Static') {
            steps {
                script {
                    bat 'C:\\Python313\\python.exe -m flake8 . --count --exit-zero > flake8-report.txt'
                    def flake8Output = readFile('flake8-report.txt').trim()
                    echo "Flake8 encontró: ${flake8Output} problemas."
                    
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
                    // Usar ruta absoluta y --exit-zero
                    bat 'C:\\Python313\\python.exe -m bandit -r . -f json -o bandit-report.json --exit-zero 2>nul'
                    def banditReport = readJSON file: 'bandit-report.json'
                    def totalIssues = banditReport.metrics.total_issues
                    echo "Bandit encontró: ${totalIssues} problemas."
                    
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
                    bat 'C:\\Python313\\python.exe -m coverage run --source=app -m pytest test\\unit'
                    bat 'C:\\Python313\\python.exe -m coverage xml -o coverage.xml'
                    bat 'C:\\Python313\\python.exe -m coverage html -d coverage_html'
                    bat 'C:\\Python313\\python.exe -m coverage report'
                }
            }
            post {
                always {
                    // Publicar resultados. Si falla esta línea, prueba con la otra opción.
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']])
                    // publishCoverage adapters: [coberturaAdapter('coverage.xml')]
                }
            }
        }

        stage('Performance') {
            steps {
                script {
                    bat 'start /B C:\\Python313\\python.exe -m flask --app app\\api run --port 5000'
                    sleep 5
                    // AJUSTA ESTA RUTA A TU INSTALACIÓN DE JMETER
                    bat 'C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t jmeter\\test-plan.jmx -l jmeter\\results.jtl -j jmeter\\jmeter.log'
                }
            }
            post {
                always {
                    bat 'taskkill /F /IM python.exe 2>nul || echo "No proceso Python para terminar"'
                    perfReport sourceDataFiles: 'jmeter\\results.jtl'
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