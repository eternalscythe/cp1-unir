pipeline {
  agent any
      stages {
          
        stage('Get Code') {
          steps {
            git url: 'https://github.com/eternalscythe/cp1-unir.git'
            echo WORKSPACE
            bat 'dir'
          }
        }
        
        stage('Unit') {
          steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                    bat '''
                        SET PYTHONPATH=%WORKSPACE%
                        pytest test\\unit
                    '''
                }
            }
        }
        
               stage('Services') {
            steps {
                script {
                    // 1. Iniciar Flask en segundo plano de forma tradicional
                    bat 'SET FLASK_APP=app\\api.py'
                    bat 'start /B flask run'
                    
                    // 2. Iniciar Wiremock de forma ROBUSTA y CAPTURAR su PID
                    //    El truco está en usar 'cmd /c' y redirigir la salida.
                    bat '''
                        cd test\\wiremock
                        cmd /c "start /B java -jar wiremock-standalone-3.13.2.jar --port 9090 --root-dir ." > ..\\..\\wiremock.log 2>&1
                        echo %ERRORLEVEL% > ..\\..\\wiremock_pid.txt
                        cd ..\\..
                    '''
                    // 3. Esperar con más tiempo (10 segundos)
                    bat 'ping -n 10 127.0.0.1 >nul'
                    // 4. Verificar que Wiremock está escuchando (DEBUG)
                    bat 'netstat -an | findstr :9090 || echo Puerto 9090 NO encontrado'
                    
                    // 5. Ejecutar las pruebas
                    bat '''
                        SET PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result.rest.xml test\\rest
                    '''
                }
                junit 'result*.xml'
            }
            post {
                always {
                    // 6. MATAR PROCESOS de forma SEGURA y ESPECÍFICA
                    script {
                        // Intentar matar Flask por nombre de proceso
                        bat 'taskkill /F /IM flask.exe 2>nul || taskkill /F /IM python.exe 2>nul || echo "No se pudo terminar Flask"'
                        // Intentar matar Wiremock usando un PID pre-guardado (más seguro que matar todos los java.exe)
                        // Si el archivo con el PID no existe, se omite.
                        bat '''
                            if exist wiremock_pid.txt (
                                for /f "tokens=*" %%i in (wiremock_pid.txt) do (
                                    taskkill /F /PID %%i 2>nul || echo No se pudo terminar Wiremock PID %%i
                                )
                                del wiremock_pid.txt 2>nul
                            )
                        '''
                    }
                }
            }
        }
        
        stage('Static') {
          steps {
            bat 'flake8 --exit-zero --format=pylint app >flake8.out'
            recordIssues tools: [flake8(name:'flake8', pattern:'flake8.out')], qualityGates: [[threshold:8, type: 'TOTAL', unstable:true], [threshold:10, type: 'TOTAL', unstable:false]]
          }
        }
        
        stage('Security') {
        steps {
            bat '''
                bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
            '''
            recordIssues tools: [pyLint(name: 'Bandit', pattern: 'bandit.out')], qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true],[threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }
        
        stage('Cobertura') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    coverage run --branch --source=app --omit=app\\_init_.py,app\\api.py -m pytest test\\unit
                    coverage xml
                '''
                recordCoverage qualityGates: [[criticality: 'NOTE', integerThreshold: 95, metric: 'LINE', threshold: 95.0], [criticality: 'ERROR', integerThreshold: 85, metric: 'LINE', threshold: 85.0], [criticality: 'NOTE', integerThreshold: 90, metric: 'BRANCH', threshold: 90.0], [criticality: 'ERROR', integerThreshold: 80, metric: 'BRANCH', threshold: 80.0]], tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']]
            }
        }
        
        stage('Performance') {
        steps {
            bat '''
                SET FLASK_APP=app\\api.py
                start flask run
                ping -n 4 127.0.0.1
                C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t C:\\DevOps\\CP1\\helloworld\\jmeter\\test-plan.jmx -f -l flask.jtl
             '''
             perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}