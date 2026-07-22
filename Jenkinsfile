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
                powershell '''
                    Write-Host "Workspace: $PWD"
                    Get-ChildItem
                    if (-not (Test-Path "pom.xml")) { Write-Error "pom.xml not found"; exit 1 }
                    Write-Host "JAVA_HOME=$env:JAVA_HOME"
                    java -version
                    mvn -version
                    mvn -B clean package
                '''
            }
        }
        stage ('Deployment') {
            steps {
                echo 'Deploy the application'
              bat '''
                    echo "Deploying the application..."
                    # Add your deployment commands here
                    copy /Y target\\onlinebookstore.war "C:\\Program Files\\Apache Software Foundation\\Tomcat 11.0\\webapps"
			Dir "C:\\Program Files\\Apache Software Foundation\\Tomcat 11.0\\webapps\\"
			timeout /t 30
			Dir "C:\\Program Files\\Apache Software Foundation\\Tomcat 11.0\\webapps\\"
			timeout /t 60
			Dir "C:\\Program Files\\Apache Software Foundation\\Tomcat 11.0\\webapps\\"
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
