pipeline {
    agent {
        label 'linux'
    }

    stages {
        stage('Get Code') {
            steps {
                echo 'Hola desde el primer stage'
                // git 'https://github.com/yurifrezzato/pruebarepo.git'
                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
            }
        }
        
        stage('Build') {
            steps {
                echo 'No hago nada'
            }
        }
        
        stage('Tests') {
            parallel {
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                export PYTHONPATH=${WORKSPACE}
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                
                stage('Service') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            sh'''
                                export FLASK_APP=app/api.py
                                flask run &
                                java -jar /home/jenkins/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                                sleep 5
                                export PYTHONPATH=${WORKSPACE}
                                pytest --junitxml=result-rest.xml test/rest
                            '''
                        }
                    }
                }
            }
        }
        
        stage('Result') {
            steps {
                junit 'result*.xml'
            }
        }
    }
}
