pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "ORACLEJDK11"
        
    }

    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin123"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vpro-maven-central"
        NEXUSIP = "172.31.84.35"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vpro-maven2-group"
        NEXUS_LOGIN = "nexuslogin"
        SONARSERVER = "sonarserver"
        SONNARSCANNER = "sonarscanner"

    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        
        stage('Unit Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        
        stage('checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        } 

        stage("sonar analysis") {
            environment {
                scannerHome = tool "${SONNARSCANNER}"
            } 

            steps {
                withSonarQubeEnv("${SONNARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                
                }   
            }
        }

        stage('quality gates')  {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }  

        stage('Upload to Nexus') {
            steps {
                nexusArtifactUploader (
                credentialsId: "${NEXUS_LOGIN}", 
                groupId: 'QA', 
                nexusUrl: "${NEXUSIP}:${NEXUSPORT}", 
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: 'vprofile-release', 
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                artifacts: [
                  [artifactId: 'vproapp',
                   classifier: '',
                   file: "target/vprofile-v2.war",
                   type: 'war']
                ]
                )
            }
        }
        
    }

    post {
    always {
        script {
            // Define variables
            def jobName = env.JOB_NAME
            def buildNumber = env.BUILD_NUMBER
            def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
            def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'

            // Create HTML email body
            def body = """
                <html>
                    <body>
                        <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                            <h2>${jobName} - Build ${buildNumber}</h2>
                            <div style="background-color: ${bannerColor}; padding: 10px;">
                                <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                            </div>
                            <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                        </div>
                    </body>
                </html>
            """

            // Send email with the generated body
            emailext(
                subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                body: body,
                to: 'boluwatifeadenle26@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
                // attachmentsPattern: 'trivy-image-report.html'
            )
        }
    }
    }

}

