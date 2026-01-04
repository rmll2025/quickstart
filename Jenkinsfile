// Variables globales
def projectPath = 'kitchensink'
def deploymentPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/deployments/'
def tempFilesPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/tmp/'
def oldFilesPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/'
def artifactName = 'kitchensink'
def tag = ''

pipeline {
    agent any

    options {
        skipStagesAfterUnstable()
    }

    stages {

        stage('Checkout') {
            steps {
                script {
                    echo " .... Checking out repository"
                    checkout scm
                }
            }
        }

        stage('Get latest tag') {
            steps {
                script {
                    echo " .... Getting latest Git tag"
                    tag = sh(
                        script: "cd ${projectPath} && git describe --tags --abbrev=0",
                        returnStdout: true
                    ).trim()

                    echo "Último tag detectado: ${tag}"
                }
            }
        }

        stage('Run tests') {
            steps {
                script {
                    echo " .... Running tests"
                    sh "cd ${projectPath} && mvn -q test"
                }
            }
        }

        stage('Build artifact') {
            steps {
                script {
                    echo " .... Building artifact"
                    sh "cd ${projectPath} && mvn -q clean install"
                }
            }
        }

        stage('Stop WildFly') {
            steps {
                script {
                    echo " .... Stopping WildFly"
                    sh "ssh sistemas@192.168.188.150 'sudo systemctl stop wildfly'"
                }
            }
        }

        stage('Backup old deployment') {
            steps {
                script {
                    echo " .... Saving old deployment"
                    sh "ssh sistemas@192.168.188.150 'sudo mv ${deploymentPath}${artifactName}.war ${oldFilesPath}'"
                }
            }
        }

        stage('Copy new WAR') {
            steps {
                script {
                    echo " .... Copying new deployment"
                    sh "scp ${projectPath}/target/${artifactName}.war sistemas@192.168.188.150:${deploymentPath}"
                }
            }
        }

        stage('Clean temp files') {
            steps {
                script {
                    echo " .... Cleaning temp files"
                    sh "ssh sistemas@192.168.188.150 'sudo rm -rf ${tempFilesPath}/*'"
                }
            }
        }

        stage('Start WildFly') {
            steps {
                script {
                    echo " .... Starting WildFly"
                    sh "ssh sistemas@192.168.188.150 'sudo systemctl start wildfly'"
                }
            }
        }
    }
}
