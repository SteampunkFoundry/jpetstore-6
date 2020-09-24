pipeline {
    agent any
    stages {
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
        stage ('Verify') {
            steps {
                sh 'mvn verify -DskipITs'
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/target/*.war', onlyIfSuccessful: true
                }
            }
        }
    }
}
