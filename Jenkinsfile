@Library('jenkinsSharedLibrarySASS@feature/initial') import gov.dhs.ice.secdevops.mavenUtility
// Only one person can use the cache at a time.
properties([disableConcurrentBuilds()])

def utils = new mavenUtility(this)
def pvc = utils.getMavenCache()
def label = "jpetstorepod-${UUID.randomUUID().toString()}"

podTemplate(
    label: label,
    containers: [
        containerTemplate(name: 'maven',
            image: 'steampunkfoundry/mvn-jdk-node:mvn3.6.3-openjdk8-node15.4.0',
            ttyEnabled: true,
            command: 'cat')
    ],
    volumes: [
        persistentVolumeClaim(mountPath: '/root/.m2/repository', claimName: pvc, readOnly: false)
    ],
    nodeSelector: 'role=workers'
) {
    node(label) {
        stage('Clone') {
            checkout(
                [
                    $class                           : 'GitSCM',
                    branches                         : scm.branches,
                    doGenerateSubmoduleConfigurations: scm.doGenerateSubmoduleConfigurations,
                    extensions                       : scm.extensions,
                    submoduleCfg                     : [],
                    userRemoteConfigs                : scm.userRemoteConfigs
                ]
            )
        }
        stage('Container') {
            withCredentials([usernamePassword(credentialsId: 'contrast-security-apptwo', passwordVariable: 'CONTRAST_SERVICEKEY', usernameVariable: 'CONTRAST_USERNAME'),
                            usernamePassword(credentialsId: 'contrast-security-apptwo-org', passwordVariable: 'CONTRAST_APIKEY', usernameVariable: 'CONTRAST_ORGUUID')]) {
                container('maven') {
                    configFileProvider([configFile(fileId: 'mavennexus', variable: 'MAVEN_CONFIG')]) {
                        stage('Deploy') {
                            sh('mvn -s ${MAVEN_CONFIG} -P tomcat90,with-contrast -Dcontrast.username=${CONTRAST_USERNAME} -Dcontrast.serviceKey=${CONTRAST_SERVICEKEY} -Dcontrast.apiKey=${CONTRAST_APIKEY} -Dcontrast.orgUuid=${CONTRAST_ORGUUID} -Dcontrast.agent.logger.stdout=true -Dcontrast.agent.java.standalone_app_name=JPetStore -Dcontrast.application.path=/ deploy -DskipITs')
                            archiveArtifacts artifacts: '**/target/dependency-check-report.*', onlyIfSuccessful: false
                            archiveArtifacts artifacts: '**/target/*.war', onlyIfSuccessful: true
                        }
                    }
                }
            }
        }
    }
}

