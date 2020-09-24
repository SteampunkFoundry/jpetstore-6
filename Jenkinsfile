podTemplate(label: 'maven',
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
                    stage ('Compile') {
                        steps {
                            sh 'mvn compile'
                        }
                    }
                    stage ('Test') {
                        steps {
                            sh 'mvn test'
                        }
                        post {
                            always {
                                junit '**/target/surefire-reports/TEST-*.xml'
                            }
                        }
                    }
                    stage ('Deploy') {
                        steps {
                            sh 'mvn deploy -DskipITs'
                        }
                        post {
                            always {
                                archiveArtifacts artifacts: '**/target/*.war', onlyIfSuccessful: true
                            }
                        }
                    }
                }
            }
        }
    }
}
