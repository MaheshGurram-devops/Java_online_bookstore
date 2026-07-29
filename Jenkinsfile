// Jenkins Pipeline for Online Bookstore Deployment
// This pipeline builds, packages, and deploys the Java bookstore application to Tomcat

pipeline {
    agent any
    
    environment {
        // Set Maven heap memory limit for build optimization
        MAVEN_OPTS = '-Xmx1024m'
        // TOMCAT_HOME can be updated if Tomcat is installed in a different location
        TOMCAT_HOME = '/opt/tomcat8'
        TOMCAT_WEBAPPS = "${TOMCAT_HOME}/webapps"
    }
    
    stages {
        // Stage 1: Clone repository from version control
        stage('Checkout') {
            steps {
                echo '========== Checking out source code =========='
                checkout scm
            }
        }
        
        // Stage 2: Build project using Maven
        stage('Build') {
            steps {
                echo '========== Building project with Maven =========='
                bat '''
                mvn -B clean package
                '''
                script {
                    // Read artifact finalName from pom.xml so WAR name is always correct
                    env.ARTIFACT_NAME = bat(
                        returnStdout: true,
                        script: '''
                            mvn -B -q -DforceStdout help:evaluate -Dexpression=project.build.finalName
                        '''
                    ).trim()
                    env.WAR_FILE = "${env.ARTIFACT_NAME}.war"
                }
            }
        }
        
        // Stage 3: Prepare WAR artifact for deployment
        stage('Prepare Artifact') {
            steps {
                echo '========== Preparing deployment artifacts =========='
                bat '''
                    if not exist output mkdir output
                    if exist target\\%WAR_FILE% (
                        copy target\\%WAR_FILE% output\\%WAR_FILE%
                        echo Successfully prepared %WAR_FILE%
                        dir output
                    ) else (
                        echo ERROR: WAR file not found at target\\%WAR_FILE%
                        exit /b 1
                    )
                '''
            }
        }
        
        // Stage 4: Deploy WAR to Tomcat application server
        stage('Deploy to Tomcat') {
            steps {
                echo '========== Deploying to Tomcat =========='
                bat '''
                    if not exist "%TOMCAT_HOME%\\bin" (
                        echo ERROR: Tomcat not found at %TOMCAT_HOME%
                        exit /b 1
                    )

                    if exist "%TOMCAT_HOME%\\bin\\shutdown.bat" (
                        call "%TOMCAT_HOME%\\bin\\shutdown.bat"
                    ) else if exist "%TOMCAT_HOME%\\bin\\shutdown.sh" (
                        call "%TOMCAT_HOME%\\bin\\shutdown.sh"
                    )

                    timeout /t 2 /nobreak >nul

                    del /q "%TOMCAT_WEBAPPS%\\*.war" 2>nul
                    if exist "%TOMCAT_WEBAPPS%\\%ARTIFACT_NAME%" rmdir /s /q "%TOMCAT_WEBAPPS%\\%ARTIFACT_NAME%"

                    copy output\\%WAR_FILE% "%TOMCAT_WEBAPPS%\\"
                    echo Deployed %WAR_FILE% to %TOMCAT_WEBAPPS%

                    if exist "%TOMCAT_HOME%\\bin\\startup.bat" (
                        call "%TOMCAT_HOME%\\bin\\startup.bat"
                    ) else if exist "%TOMCAT_HOME%\\bin\\startup.sh" (
                        call "%TOMCAT_HOME%\\bin\\startup.sh"
                    )

                    timeout /t 3 /nobreak >nul
                    echo Tomcat started successfully
                '''
            }
        }
    }
    
    // Post-build actions
    post {
        // Archive WAR file regardless of build status
        always {
            echo '========== Archiving artifacts =========='
            archiveArtifacts artifacts: 'output/*.war', fingerprint: true, allowEmptyArchive: false
        }
        
        // Success notification
        success {
            echo '========== Deployment successful =========='
            echo "Application available at: http://localhost:8080/${ARTIFACT_NAME}"
        }
        
        // Failure notification
        failure {
            echo '========== Deployment failed =========='
            echo 'Please check the logs above for details'
        }
    }
}
