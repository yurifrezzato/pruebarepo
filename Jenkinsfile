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

        stage('Unit') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh'''
                        export PYTHONPATH=${WORKSPACE}
                        pytest --junitxml=result-unit.xml test/unit
                    '''
                    junit 'result-unit.xml'
                }
            }
        }
            
        stage('Service') {
            steps {
                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    sh'''
                        export FLASK_APP=app/api.py
                        flask run &
                        java -jar /var/jenkins_home/wiremock-standalone-3.10.0.jar --port 9090 --root-dir ${WORKSPACE}/test/wiremock &
                        sleep 5
                        export PYTHONPATH=${WORKSPACE}
                        pytest --junitxml=result-rest.xml test/rest
                    '''
                    junit 'result-rest.xml'
                }
            }
        }

        stage('Coverage') {
            steps {
                sh'''
                    coverage run --branch --source=app --omit=app/__init__.py,app/api.py -m pytest test/unit
                    coverage xml
                '''

                catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
                    cobertura coberturaReportFile: 'coverage.xml', conditionalCoverageTargets: '100,0,85', lineCoverageTargets: '100,0,90'
                }
            }
        }

        // stage('Static') {
        //     steps {
        //             sh'''
        //                 flake8 --format=pylint app >falke8.out
        //             '''
        //             junit 'result-rest.xml'

        //         catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE') {
        //             recordIssues tools: [flake8(name: 'Flake8', pattern: 'flake8.out')], qualityGates: [[threshold: 10, type: 'TOTAL', unestable: true], [threshold: 11, type: 'TOTAL', unestable: false]]
        //         }
        //     }
        // }
    }
}
