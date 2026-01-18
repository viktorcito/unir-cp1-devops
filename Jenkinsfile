pipeline {
    agent any
    environment {
        PYTHON = '"C:\\Program Files\\Python314\\python.exe"'
    }
    stages {
        stage('Get Code') {
            steps {
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
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        "C:\\Program Files\\Python314\\python.exe" -m flake8 app --exit-zero
                    '''
                }
            }
        }
        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        "C:\\Program Files\\Python314\\python.exe" -m bandit -r app --exit-zero
                    '''
                }
            }
        }
        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    bat '''
                        set PYTHONPATH=%WORKSPACE%
                        "C:\\Program Files\\Python314\\python.exe" -m coverage run --branch --source=app -m pytest test\\unit
                        "C:\\Program Files\\Python314\\python.exe" -m coverage report
                        "C:\\Program Files\\Python314\\python.exe" -m coverage xml -o coverage.xml
                    '''
                    recordCoverage(tools: [[parser: 'COBERTURA', pattern: 'coverage.xml']])
                }
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
