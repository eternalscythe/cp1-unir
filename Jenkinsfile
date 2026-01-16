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
            bat '''
                SET FLASK_APP=app\\api.py
                start flask run
                start java -jar test\\wiremock\\wiremock-standalone-3.13.2.jar --port 9090 --root-dir test\\wiremock
                timeout /t 10 /nobreak
                SET PYTHONPATH=%WORKSPACE%
                pytest --junitxml=result.rest.xml test\\rest
            '''
            junit 'result*.xml'
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
                C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t C:\\DevOps\\CP1\\helloworld\\jmeter\\test-plan.jmx" -f -l flask.jtl
             '''
             perfReport sourceDataFiles: 'flask.jtl'
            }
        }
    }
}