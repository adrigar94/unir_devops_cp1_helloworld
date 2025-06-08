pipeline {
    agent { label 'linux' }

    stages {
        stage('Get Code') {
            steps {
                echo 'Downloading code'
                git branch: 'master', url: 'https://github.com/adrigar94/unir_devops_cp1_helloworld.git'
                sh 'ls -la'
                stash name: 'code', includes: '**/*,.*'
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
            }
            post {
                always {
                    deleteDir()
                }
            }
        }

        stage('Unit') {
                    steps {
                        echo WORKSPACE
                        sh 'whoami'
                        sh 'hostname'
                        unstash 'code'
                        sh '''
                            export PYTHONPATH=$WORKSPACE
                            coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit --junitxml=result-unit.xml
                        '''
                        junit 'result-unit.xml'
                        stash name: 'coverage-report', includes: '**/.coverage'
                    }
                    post {
                        always {
                            deleteDir()
                        }
                    }
        }

        stage('Rest') {
            steps {
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
                unstash 'code'
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        export FLASK_APP=app/api.py
                        export FLASK_ENV=development
                        flask run &
                        java -jar /opt/wiremock/wiremock-standalone-3.13.0.jar --port 9090 --root-dir test/wiremock &

                        echo "Esperando a que Flask (5000) y WireMock (9090) estén disponibles..."
                        for i in {1..15}; do
                            if curl -s http://localhost:5000/ > /dev/null && curl -s http://localhost:9090/__admin > /dev/null; then
                                echo "Servicios disponibles."
                                break
                            fi
                            echo "Esperando servicios... intento $i"
                            sleep 1
                            if [ "$i" -eq 15 ]; then
                                echo "Timeout esperando a los servicios"
                                exit 1
                            fi
                        done

                        pytest --junitxml=result-rest.xml  test/rest
                    '''
                }
                junit 'result-rest.xml'
            }
            post {
                always {
                    deleteDir()
                }
            }
        }

        stage('Coverage') {
            steps {
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
                unstash 'code'
                unstash 'coverage-report'
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    coverage xml -o cobertura.xml
                '''
                recordCoverage(
                    tools: [[parser: 'COBERTURA']],
                    sourceCodeRetention: 'EVERY_BUILD',
                    qualityGates: [
                        [threshold: 85.0, metric: 'LINE', baseline: 'PROJECT', failBuild: true],
                        [threshold: 95.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                        [threshold: 80.0, metric: 'BRANCH', baseline: 'PROJECT', failBuild: true],
                        [threshold: 90.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]
                    ]
                )
            }
            post {
                always {
                    deleteDir()
                }
            }
        }

        stage('Static') {
            steps {
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
                unstash 'code'
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    flake8 --exit-zero --format=pylint app > static.out
                '''
                recordIssues tools: [flake8(pattern: 'static.out')],
                    qualityGates: [
                        [threshold: 8, type: 'TOTAL', unstable: true],
                        [threshold: 10, type: 'TOTAL', criticality: 'ERROR'],
                    ]
            }
            post {
                always {
                    deleteDir()
                }
            }
        }



        stage('Security') {
            steps {
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
                unstash 'code'
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    bandit -r . -f custom -o security.out --msg-template "{abspath}:{line}: [{test_id}] {msg}" || true
                '''
                recordIssues tools: [pyLint(pattern: 'security.out')],
                    qualityGates: [
                        [threshold: 2, type: 'TOTAL', unstable: true],
                        [threshold: 4, type: 'TOTAL', criticality: 'ERROR'],
                    ]
            }
            post {
                always {
                    deleteDir()
                }
            }
        }


        stage('Performance') {
            steps {
                echo WORKSPACE
                sh 'whoami'
                sh 'hostname'
                unstash 'code'
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        export PYTHONPATH=$WORKSPACE
                        export FLASK_APP=app/api.py
                        export FLASK_ENV=development
                        flask run &
                        java -jar /opt/wiremock/wiremock-standalone-3.13.0.jar --port 9090 --root-dir test/wiremock &

                        echo "Esperando a que Flask (5000) y WireMock (9090) estén disponibles..."
                        for i in {1..15}; do
                            if curl -s http://localhost:5000/ > /dev/null && curl -s http://localhost:9090/__admin > /dev/null; then
                                echo "Servicios disponibles."
                                break
                            fi
                            echo "Esperando servicios... intento $i"
                            sleep 1
                            if [ "$i" -eq 15 ]; then
                                echo "Timeout esperando a los servicios"
                                exit 1
                            fi
                        done
                    '''
                }
                sh '''
                    export PYTHONPATH=$WORKSPACE
                    /opt/jmeter/bin/jmeter -n -t test/jmeter/flask.jmx --forceDeleteResultFile --logfile performance.jtl
                '''
                perfReport sourceDataFiles: 'performance.jtl'
            }
            post {
                always {
                    deleteDir()
                }
            }
        }
    }
}
