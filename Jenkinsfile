@Library('jenkins-pipeline-shared-libraries')_

pipeline {
    agent {
        label 'submarine-static || kie-rhel7'
    }
    tools {
        maven 'kie-maven-3.5.4'
        jdk 'kie-jdk1.8'
    }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '', numToKeepStr: '10')
    }
    stages {
        stage('Initialize') {
            steps {
                sh 'printenv'
            }
        }
        stage('Build submarine-bom') {
            steps {
                timeout(15) {
                    dir("submarine-bom") {
                        script {
                            githubscm.checkoutIfExists('submarine-bom', "$CHANGE_AUTHOR", "$CHANGE_BRANCH", 'kiegroup', "$CHANGE_TARGET")
                            maven.runMavenWithSubmarineSettings('clean install', true)
                        }
                    }
                }
            }
        }
        stage('Build submarine-runtimes') {
            steps {
                timeout(30) {
                    dir("submarine-runtimes") {
                        script {
                            githubscm.checkoutIfExists('submarine-runtimes', "$CHANGE_AUTHOR", "$CHANGE_BRANCH", 'kiegroup', "$CHANGE_TARGET")
                            maven.runMavenWithSubmarineSettings('clean install', true)
                        }
                    }
                }
            }
        }
        stage('Build submarine-cloud') {
            steps {
                timeout(30) {
                    script {
                        maven.runMavenWithSubmarineSettings('clean install', false)
                    }
                }
            }
        }
        // Currently there are no tests in submarine-cloud
//        stage('Publish test results') {
//            steps {
//                junit '**/target/surefire-reports/**/*.xml'
//            }
//        }
    }
    post {
        unstable {
            script {
                mailer.sendEmailFailure()
            }
        }
        failure {
            script {
                mailer.sendEmailFailure()
            }
        }
        always {
            cleanWs()
        }
    }
}