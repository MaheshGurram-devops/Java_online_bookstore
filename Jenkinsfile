pipeline {
    agent any
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