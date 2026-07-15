pipeline {
    agent any
    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }
    stages {
        stage('Sourcecode checkout') {
            steps {
                echo 'Clone Java Online Bookstore repository'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Build the project'
                sh '''
                    echo "Workspace: $(pwd)"
                    ls -la
                    test -f pom.xml || { echo 'pom.xml not found'; exit 1; }
                    echo "JAVA_HOME=${JAVA_HOME:-}"
                    java -version
                    mvn -version
                    mvn -B clean package
                '''
            }
        }
    }
    post {
        always {
            echo 'Archieving build artifacts'
            archiveArtifacts artifacts: '**/target/*.war', fingerprint: true, allowEmptyArchive: true
        }
        success {
            echo 'Build completed successfully'
        }
        failure {
            echo 'Build failed'
        }
    }
}