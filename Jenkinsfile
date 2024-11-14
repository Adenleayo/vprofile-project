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
        SONNARSCANNER = "sonar6.2"

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
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}"
                artifacts: [
                    [
                        artifactId: 'vproapp',
                        classifier: '',
                        file: "target/vprofile-v2.war",
                        type: 'war'
                    ]
                ], 
                )
            }
        }
    }
}