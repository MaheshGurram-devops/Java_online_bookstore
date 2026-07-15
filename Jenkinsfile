pipeline {
    agent any
    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }
    stages {
        stage('Sourcecode checkout') {
            steps {
                echo 'Clone Java Online Bookstore repository'
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        rm -rf workspace-copy
                        git clone https://$GIT_USER:$GIT_PASS@github.com/umamaheshmgangadhar-byte/Java_online_bookstore.git workspace-copy
                        cd workspace-copy
                        pwd
                        ls -la
                    '''
                }
            }
        }
        stage('Build') {
            steps {
                echo 'Build the project'
                sh '''
                    echo "Workspace: $(pwd)"
                    ls -la
                    cd workspace-copy
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