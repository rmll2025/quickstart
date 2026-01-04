// Variables globales
def projectPath = 'quickstart/kitchensink'
def deploymentPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/deployments/'
def tempFilesPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/tmp/'
def oldFilesPath = '/opt/wildfly/wildfly-38.0.1.Final/standalone/'
def artifactName = 'kitchensink'
def tag = ''

pipeline {
    agent any

    // Si falla alguna etapa, no continua
	options {
        skipStagesAfterUnstable()
    }

    stages {

        stage('Clonado') {
            steps {
                script {
                    echo " .... Clonando repositorio"
                    checkout scm
                }
            }
        }

        stage('Ultimo Tag') {
            steps {
                script {
                    echo " .... Obteniendo el último Git tag"
                    tag = sh(
                        script: "cd ${projectPath} && git describe --tags --abbrev=0",
                        returnStdout: true
                    ).trim()

                    echo "Último tag detectado: ${tag}"
                }
            }
        }

        stage('Comprobaciones') {
            steps {
                script {
                    echo " .... Ejecutando comprobaciones"
                    sh "cd ${projectPath} && mvn -q test"
                }
            }
        }

        stage('Generar') {
            steps {
                script {
                    echo " .... Generando artefacto"
                    sh "cd ${projectPath} && mvn -q clean install"
                }
            }
        }

        stage('Detener WildFly') {
            steps {
                script {
                    echo " .... Deteniendo WildFly"
                    sh "ssh sistemas@192.168.188.150 'sudo systemctl stop wildfly'"
                }
            }
        }

        stage('Copia de Seguridad') {
            steps {
                script {
                    sh """
						ssh sistemas@192.168.188.150 '
						# Si existe un WAR previo, lo movemos al directorio oldFilesPath
						if ls ${deploymentPath}${artifactName}*.war 1>/dev/null 2>&1; then
							sudo mv ${deploymentPath}${artifactName}*.war ${oldFilesPath}
						else
							echo "No se ha encontrado versión previa del artefacto, omitimos copia de seguridad"
						fi
						'
					"""
                }
            }
        }

        stage('Copia del nuevo artefacto') {
            steps {
                script {
                    echo " .... Copiando el nuevo artefacto generado"
                    sh "scp ${projectPath}/target/${artifactName}.war sistemas@192.168.188.150:${deploymentPath}"
                }
            }
        }

        stage('Limpieza') {
            steps {
                script {
                    echo " .... Limpiando archivos temporales"
                    sh "ssh sistemas@192.168.188.150 'sudo rm -rf ${tempFilesPath}/*'"
                }
            }
        }

        stage('Arrancar WildFly') {
            steps {
                script {
                    echo " .... Arrancando WildFly"
                    sh "ssh sistemas@192.168.188.150 'sudo systemctl start wildfly'"
                }
            }
        }
    }
}
