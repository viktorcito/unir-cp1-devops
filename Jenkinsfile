pipeline {
    agent any
    environment {
        PYTHON = '"C:\\Program Files\\Python314\\python.exe"'
    }
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
                        "C:\\Program Files\\Python314\\python.exe" -m pytest test\\unit --junitxml=result-unit.xml
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
                        start "" /B cmd /c "C:\\Program Files\\Python314\\python.exe" -m flask run
                        start "" /B java -jar C:\\wiremock\\wiremock-standalone.jar --port 9090 --root-dir C:\\wiremock
                        ping 127.0.0.1 -n 6 > nul
                        set PYTHONPATH=%WORKSPACE%
                        "C:\\Program Files\\Python314\\python.exe" -m pytest test\\rest --junitxml=result-rest.xml
                    '''
                }
            }
        }
        stage('Static') {
            steps {
                bat '''
                    "C:\\Program Files\\Python314\\python.exe" -m flake8 --format=checkstyle --output-file=flake8-result.xml app || exit 0
                '''
                recordIssues tools: [checkStyle(pattern: 'flake8-result.xml', reportEncoding: 'UTF-8')], 
                    qualityGates: [[threshold: 8, type: 'TOTAL', unstable: true], [threshold: 10, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Security') {
            steps {
                bat '''
                    "C:\\Program Files\\Python314\\python.exe" -m bandit -r app -f custom -o bandit-result.txt --msg-template "{abspath}:{line}: [{test_id}] {msg}" || exit 0
                '''
                recordIssues tools: [pyLint(pattern: 'bandit-result.txt', reportEncoding: 'UTF-8')],
                    qualityGates: [[threshold: 2, type: 'TOTAL', unstable: true], [threshold: 4, type: 'TOTAL', unstable: false]]
            }
        }
        stage('Coverage') {
            steps {
                bat '''
                    set PYTHONPATH=%WORKSPACE%
                    "C:\\Program Files\\Python314\\python.exe" -m coverage run --branch --source=app -m pytest test\\unit
                    "C:\\Program Files\\Python314\\python.exe" -m coverage xml -o coverage.xml
                '''
                cobertura coberturaReportFile: 'coverage.xml',
                    conditionalCoverageTargets: '80, 0, 90',
                    lineCoverageTargets: '85, 0, 95'
            }
        }
        stage('Performance') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set FLASK_APP=app\\api.py
                        start "" /B cmd /c "C:\\Program Files\\Python314\\python.exe" -m flask run
                        ping 127.0.0.1 -n 4 > nul
                        C:\\jmeter\\apache-jmeter-5.6.3\\bin\\jmeter -n -t test\\jmeter\\flask.jmx -l result-performance.jtl
                    '''
                    perfReport sourceDataFiles: 'result-performance.jtl'
                }
            }
        }
    }
    post {
        always {
            junit 'result-*.xml'
        }
    }
}

