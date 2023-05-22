pipeline {
    agent any

    options {
        gitLabConnection('gitlab')
    }

    stages {
        stage("build") {
            agent {
                docker {
                    image 'python:3.6'
                    args '-u root'
                }
            }
            steps {
                sh """
                pip3 install --user virtualenv
                python3 -m virtualenv env
                . env/bin/activate
                pip3 install -r requirements.txt
                python3 manage.py check
                """
            }
        }
        stage("test") {
            agent {
                docker {
                    image 'python:3.6'
                    args '-u root'
                }
            }
            steps {
                sh """
                pip3 install --user virtualenv
                python3 -m virtualenv env
                . env/bin/activate
                pip3 install -r requirements.txt
                python3 manage.py test taskManager
                """
            }
        }
        stage("integration") {
            parallel {
                stage("nikto") {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh "docker run -v \$(pwd):/tmp --rm hysnsec/nikto -h https://prod-kyqozhov.lab.practical-devsecops.training -o /tmp/nikto-output.xml"
                        }
                    }
                }
                stage("sslyze") {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh "docker run -v \$(pwd):/tmp --rm hysnsec/sslyze prod-kyqozhov.lab.practical-devsecops.training:443 --json_out /tmp/sslyze-output.json"
                        }
                    }
                }
                stage("nmap") {
                    steps {
                        catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                            sh "docker run -v \$(pwd):/tmp --rm hysnsec/nmap prod-kyqozhov -oX /tmp/nmap-output.xml"
                        }
                    }
                }
            }
        }
        stage("prod") {
            steps {
                echo "This is a deploy step"
            }
        }
    }
    post {
        failure {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'failed'
        }
        unstable {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'failed'
        }
        success {
            updateGitlabCommitStatus name: STAGE_NAME, state: 'success'
        }
        always {
            archiveArtifacts artifacts: '*.json,*.xml', fingerprint: true
            deleteDir()                     // clean up workspace
            dir("${WORKSPACE}@tmp") {       // clean up tmp directory
                deleteDir()
            }
            dir("${WORKSPACE}@script") {    // clean up script directory
                deleteDir()
            }
        }
    }
}
