pipeline {
    agent { label 'linux' }

    stages {
        stage('Get Code') {
            steps {
                echo 'Downloading code'
                git branch: 'develop', url: 'https://github.com/adrigar94/unir_devops_cp1_helloworld.git'
                sh 'ls -la'
                echo WORKSPACE
            }
        }
        
        stage('Build') {
            steps {
                echo 'Build code'
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        sh '''
                            export PYTHONPATH=$WORKSPACE
                            pytest --junitxml=result-unit.xml test/unit
                        '''
                    }
                }
                
                stage('Rest') {
                    steps {
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
                        
                    }
                }
            }
        }

        stage('Results') {
            steps {
                junit 'result*.xml'
            }
        }
        

    }
}
