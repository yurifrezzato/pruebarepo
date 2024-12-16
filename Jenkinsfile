pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                echo 'Hola desde el primer stage'
                git 'https://github.com/yurifrezzato/pruebarepo.git'
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

                            // run services
                            sh''' 
                                export FLASK_APP=app/api.py
                                flask run &
                                java -jar /var/jenkins_home/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                            '''

                            // evaluate when services are up and running
                            script {
                                int w_port = 9090; //wiremock port
                                int f_port = 5000; //flask port
                                int w_port_out = 1;
                                int f_port_out = 1;

                                while(w_port_out!=0 || f_port_out!=0) {
                                    sleep 1;
                                    w_port_out = sh returnStatus: true, script: "netstat -tuplen | grep ${w_port}";
                                    f_port_out = sh returnStatus: true, script: "netstat -tuplen | grep ${f_port}";
                                    println "w_port_out: ${w_port_out}";
                                    println "f_port_out: ${f_port_out}";
                                }
                            }

                            // execute tests
                            sh'''
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
    post {
        cleanup {
            cleanWs();
        }
    }
}
