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
                powershell '''
                mvn -B clean package
                '''
                script {
                    // Read artifact finalName from pom.xml so WAR name is always correct
                    env.ARTIFACT_NAME = powershell(
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
                powershell '''
                    $outputDir = "output"
                    New-Item -ItemType Directory -Force -Path $outputDir | Out-Null

                    $sourceWar = "target/$env:WAR_FILE"
                    if (Test-Path $sourceWar) {
                        Copy-Item $sourceWar -Destination "$outputDir/$env:WAR_FILE" -Force
                        Write-Host "Successfully prepared $env:WAR_FILE"
                        Get-ChildItem $outputDir
                    } else {
                        Write-Error "WAR file not found at $sourceWar"
                        exit 1
                    }
                '''
            }
        }
        
        // Stage 4: Deploy WAR to Tomcat application server
        stage('Deploy to Tomcat') {
            steps {
                echo '========== Deploying to Tomcat =========='
                powershell '''
                    if (-not (Test-Path "$env:TOMCAT_HOME/bin")) {
                        Write-Error "Tomcat not found at $env:TOMCAT_HOME"
                        exit 1
                    }

                    $tomcatBin = Join-Path $env:TOMCAT_HOME "bin"
                    $shutdownScript = Join-Path $tomcatBin "shutdown.sh"
                    $startupScript = Join-Path $tomcatBin "startup.sh"

                    if (Test-Path $shutdownScript) {
                        & $shutdownScript 2>$null
                    } elseif (Test-Path (Join-Path $tomcatBin "shutdown.bat")) {
                        & (Join-Path $tomcatBin "shutdown.bat") 2>$null
                    }

                    Start-Sleep -Seconds 2

                    Get-ChildItem -Path $env:TOMCAT_WEBAPPS -Filter "*.war" | Remove-Item -Force -ErrorAction SilentlyContinue
                    Remove-Item -Path (Join-Path $env:TOMCAT_WEBAPPS $env:ARTIFACT_NAME) -Recurse -Force -ErrorAction SilentlyContinue

                    Copy-Item "output/$env:WAR_FILE" -Destination $env:TOMCAT_WEBAPPS -Force
                    Write-Host "Deployed $env:WAR_FILE to $env:TOMCAT_WEBAPPS"

                    if (Test-Path $startupScript) {
                        & $startupScript
                    } elseif (Test-Path (Join-Path $tomcatBin "startup.bat")) {
                        & (Join-Path $tomcatBin "startup.bat")
                    }

                    Start-Sleep -Seconds 3
                    Write-Host "Tomcat started successfully"
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
