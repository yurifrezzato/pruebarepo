pipeline {
    agent any
    options {
        skipDefaultCheckout true
    }

    stages {
        stage('Get Code') {
            steps {
                echo 'Hola desde el primer stage';

                echo 'Node info';
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution';
                git 'https://github.com/yurifrezzato/pruebarepo.git';
                sh 'ls -la';
                echo "WORKSPACE: ${WORKSPACE}";

                stash name: "github_code", includes: "**";
            }
        }
        
        stage('Build') {
            steps {
                echo 'Node info';
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution';
                echo 'No hago nada';
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
                            echo 'Node info';
                            sh '''
                                whoami
                                hostname
                            '''

                            echo 'Stage execution';
                            unstash "github_code";
                            sh'''
                                export PYTHONPATH=${WORKSPACE}
                                pytest --junitxml=result-unit.xml test/unit
                                
                            '''
                        }
                    }
                    post {
                        cleanup {
                            cleanWs();
                        }
                        success {
                            stash name: "unit_pytest", includes: "result-unit.xml";
                        }
                    }
                }
                
                stage('Service') {
                    agent {
                        label 'agent2'
                    }
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                            echo 'Node info';
                            sh '''
                                whoami
                                hostname
                            '''

                            echo 'Stage execution';
                            unstash "github_code";
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
                    post {
                        cleanup {
                            cleanWs();
                        }
                        success {
                            stash name: "service_pytest", includes: "result-rest.xml";
                        }
                    }
                }
            }
        }
        
        stage('Result') {
            steps {
                echo 'Node info';
                sh '''
                    whoami
                    hostname
                '''

                echo 'Stage execution';
                unstash "unit_pytest";
                unstash "service_pytest";
                junit 'result*.xml';
            }
        }
    }
    post {
        cleanup {
            cleanWs();
        }
    }
}
