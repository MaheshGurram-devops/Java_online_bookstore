pipeline {
    agent any
    stages {
        stage ('Sourcecode checkout') {
            steps {
                echo "Clone Java Online Bookstore repository"
                checkout scm 
            }
        }
        stage ('Build') {
            steps {
                echo "Build the project"
                sh 'mvn clean package'
            }
        }
    }
    post {
        always {
            echo "Archieving build artifacts"
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true, allowEmptyArchive: true
        }
        success {
            echo "Build completed successfully"
        }
        failure {
            echo "Build failed"
        }
    }
}