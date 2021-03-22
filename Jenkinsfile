@Library('jenkinsSharedLibrarySASS@feature/initial') import gov.dhs.ice.secdevops.mavenUtility
// Only one person can use the cache at a time.
properties([disableConcurrentBuilds()])


def utils = new mavenUtility(this)
def pvc = utils.getMavenCache()
def label = "jpetstorepod-${UUID.randomUUID().toString()}"
println("Here I am")

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
          stage('tester'){
              echo "PVC is: ${pvc}"
          }

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
                container('maven') {
                    configFileProvider([configFile(fileId: 'mavennexus', variable: 'MAVEN_CONFIG')]) {
                        stage('Compile') {
                            sh('mvn -s ${MAVEN_CONFIG} -B compile')
                        }
                        stage('SBOM') {
                            sh('mvn -s ${MAVEN_CONFIG} -B cyclonedx:makeAggregateBom')
                            archiveArtifacts artifacts: '**/target/bom.*', onlyIfSuccessful: false
                        }
                        stage('Test') {
                            sh('mvn -s ${MAVEN_CONFIG} -B test')
                            junit '**/target/surefire-reports/TEST-*.xml'
                            jacoco(
                                execPattern: 'target/*.exec',
                                classPattern: 'target/classes',
                                sourcePattern: 'src/main/java',
                                exclusionPattern: 'src/test*'
                            )
                        }
                        stage ('SonarQube analysis') {
                            withSonarQubeEnv('sonarqube') {
                                sh 'mvn -s ${MAVEN_CONFIG} -B sonar:sonar'
                            }
                        }
                        stage ('SonarQube quality gate') {
                            timeout(time: 10, unit: 'MINUTES') {
                                waitForQualityGate abortPipeline: true
                            }
                        }
                        stage('Deploy') {
                            sh('mvn -s ${MAVEN_CONFIG} -B deploy -DskipITs')
                            archiveArtifacts artifacts: '**/target/dependency-check-report.*', onlyIfSuccessful: false
                            archiveArtifacts artifacts: '**/target/*.war', onlyIfSuccessful: true
                        }
                    }
                }
        }
    }
}

