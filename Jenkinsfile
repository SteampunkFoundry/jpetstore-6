podTemplate(
    label: 'maven',
    containers: [
        containerTemplate(name: 'maven',
                image: 'maven:3.6.3-jdk-8',
                ttyEnabled: true,
                command: 'cat')
        ]
) {
    node('maven') {
        stage('Container') {
            container('maven') {
                stage('Compile') {
                    sh('mvn compile')
                }
                stage('Test') {
                    sh('mvn test')
                    junit '**/target/surefire-reports/TEST-*.xml'
                }
                stage('Deploy') {
                    sh('mvn deploy -DskipITs')
                    archiveArtifacts artifacts: '**/target/*.war', onlyIfSuccessful: true
                }
            }
        }
    }
}
