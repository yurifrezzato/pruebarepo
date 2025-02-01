pipeline {
    agent any

    stages {
        stage('Get Code') {
            steps {
                echo 'Hola desde el primer stage'
                checkout scmGit(
                    branches: [[
                        name: 'feature_fix_coverage'
                    ]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/yurifrezzato/pruebarepo.git'
                    ]]
                )

                sh 'ls -la'
                echo "WORKSPACE: ${WORKSPACE}"
            }
        }

        stage('Unit with Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest --junitxml=result-unit.xml test/unit
                    '''
                    junit 'result-unit.xml'
                }
            }
        }

        stage('Service') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        export FLASK_APP=app/api.py
                        flask run &
                        java -jar /var/jenkins_home/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                    '''

                    check_flask();
                    check_wiremock();

                    sh '''
                        export PYTHONPATH=${WORKSPACE}
                        pytest --junitxml=result-rest.xml test/rest
                    '''
                    junit 'result-rest.xml'
                }
            }
        }

        stage('Coverage') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m coverage xml
                    '''

                    cobertura coberturaReportFile: 'coverage.xml', lineCoverageTargets: '100,85,95', conditionalCoverageTargets: '100,80,90', onlyStable: false;
                }
            }
        }

        stage('Static') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m flake8 --exit-zero --format=pylint app > flake8.out
                    '''

                    recordIssues tools:
                        [flake8(name: 'Flake8', pattern: 'flake8.out')],
                        qualityGates: [
                            [threshold: 8, type: 'TOTAL', unstable: true],
                            [threshold: 10, type: 'TOTAL', unstable: false]
                        ]
                }
            }
        }

        stage('Security') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        python3 -m bandit --exit-zero -r . -f custom -o bandit.out --msg-template "{abspath}:{line}: [{test_id}] {msg}"
                    '''

                    recordIssues tools:
                        [pyLint(name: 'Bandit', pattern: 'bandit.out')],
                        qualityGates: [
                            [threshold: 2, type: 'TOTAL', unstable: true],
                            [threshold: 4, type: 'TOTAL', unstable: false]
                        ]
                }
            }
        }

        stage('Performance') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh '''
                        export FLASK_APP=app/api.py
                        flask run &
                    '''

                    check_flask();

                    sh '''
                        /var/jenkins_home/apache-jmeter-5.6.3/bin/jmeter.sh -n -t test/jmeter/flask.jmx -f -l flask.jtl
                    '''

                    perfReport sourceDataFiles: 'flask.jtl'
                }
            }
        }
    }
    post {
        cleanup {
            cleanWs();
        }
    }
}

def check_wiremock() {
    println "### Checking Wiremock Service ###"
    int w_port = 9090;
    int w_port_out = 1;
    while(w_port_out!=0) {
        sleep 1;
        w_port_out = sh returnStatus: true, script: "netstat -tuplen | grep ${w_port}";
    }
    println "### End checking Wiremock Service. RC: ${w_port_out} ###";
}

def check_flask() {
    println "### Checking Flask Service ###"
    int f_port = 5000;
    int f_port_out = 1;
    while(f_port_out!=0) {
        sleep 1;
        f_port_out = sh returnStatus: true, script: "netstat -tuplen | grep ${f_port}";
    }
    println "### End checking Flask Service. RC: ${f_port_out} ###"
}
