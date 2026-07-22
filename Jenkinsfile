pipeline {
    agent {
		label 'Linux-bookstore'
	}
    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }
    stages {
        stage('Sourcecode checkout') {
            steps {
                echo 'Checkout repository using Jenkins Git credentials'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/umamahesh-V1']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/MaheshGurram-devops/Java_online_bookstore.git',
                        credentialsId: 'Github-credentials-Uma'
                    ]]
                ])
            }
        }
        stage('Build') {
            steps {
                echo 'Build the project'
                sh '''
                    echo "Workspace: $pwd"
                    ls -l
                    java -version
                    mvn -version
                    mvn -B clean package
                '''
            }
        }
        stage ('Deployment') {
            steps {
                echo 'Deploy the application'
              sh '''
                    echo "Deploying the application..."
                    	cp -rf target/onlinebookstore.war /opt/tomcat/webapps/
			ls -l /opt/tomcat/webapps/
			sleep 10
			
                '''
            }
        }
    }
    post {
        always {
            echo 'Archieving build artifacts'
            archiveArtifacts artifacts: '**/target/*.war, **/target/*executable.jar', fingerprint: true, allowEmptyArchive: true
        }
        success {
            echo 'Build completed successfully'
        }
        failure {
            echo 'Build failed'
        }
    }
}
