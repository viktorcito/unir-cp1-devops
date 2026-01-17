pipeline {
    agent any
    stages {
        stage('Get Code') {
            steps {
                // Obtener código del repositorio
                git 'https://github.com/viktorcito/unir-cp1-devops.git'
            }
        }
        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        py -m pytest test\\unit --junitxml=result-unit.xml
                    '''
                }
            }
        }
        stage('Rest') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set FLASK_APP=app\\api.py
                        set FLASK_ENV=development
                        start /B py -m flask run
                        start /B java -jar C:\\wiremock\\wiremock-standalone.jar --port 9090 --root-dir C:\\wiremock
                        timeout /t 5
                        set PYTHONPATH=%WORKSPACE%
                        py -m pytest test\\rest --junitxml=result-rest.xml
                    '''
                }
            }
        }
        stage('Static') {
            steps {
                bat '''
                    py -m flake8 --format=checkstyle app > flake8-result.xml || exit 0
                '''
                recordIssues tools: [checkStyle(pattern: 'flake8-result.xml', reportEncoding: 'UTF-8')], 
                    qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unhealthy: true]]
            }
        }
        stage('Security') {
            steps {
                bat '''
                    py -m bandit -r app -f custom -o bandit-result.txt --msg-template "{abspath}:{line}: [{test_id}] {msg}" || exit 0
                '''
                recordIssues tools: [pyLint(pattern: 'bandit-result.txt', reportEncoding: 'UTF-8')],
                    qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unhealthy: true]]
            }
        }
        stage('Coverage') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    py -m coverage run --branch --source=app -m pytest test\\unit
                    py -m coverage xml -o coverage.xml
                '''
                cobertura coberturaReportFile: 'coverage.xml',
                    conditionalCoverageTargets: '80, 0, 90',
                    lineCoverageTargets: '85, 0, 95'
            }
        }
        stage('Performance') {
            steps {
                bat '''
                    set FLASK_APP=app\\api.py
                    start /B py -m flask run
                    timeout /t 3
                    C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -l result-performance.jtl
                '''
                perfReport sourceDataFiles: 'result-performance.jtl'
            }
        }
    }
    post {
        always {
            junit 'result-*.xml'
            cleanWs()
        }
    }
}
