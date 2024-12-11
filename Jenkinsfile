pipeline {
    agent none

    stages {
        stage('Get Code') {
            agent any
            steps {
                echo 'Hola desde el primer stage'

                echo 'Node info'
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution'
                // git 'https://github.com/yurifrezzato/pruebarepo.git'
                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
            }
        }
        
        stage('Build') {
            agent any
            steps {
                echo 'Node info'
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution'
                echo 'No hago nada'
            }
        }
        
        stage('Tests') {
            parallel {
                stage('Unit') {
                    agent {
                        label 'agent1'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            echo 'Node info'
                            sh '''
                                whoami
                                hostname
                            '''

                            echo 'Stage execution'
                            sh'''
                                export PYTHONPATH=${WORKSPACE}
                                pytest --junitxml=result-unit.xml test/unit
                            '''
                        }
                    }
                }
                
                stage('Service') {
                    agent {
                        label 'agent2'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            echo 'Node info'
                            sh '''
                                whoami
                                hostname
                            '''

                            echo 'Stage execution'
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
            agent any
            steps {
                echo 'Node info'
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution'
                junit 'result*.xml'
            }
        }
    }
    post {
        always {
            cleanWs() 
        }
    }
}
