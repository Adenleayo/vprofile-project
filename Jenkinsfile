pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OPENJDK17"
        
    }
    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin123"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vpro-maven-central"
        NEXUS_IP = "172.31.84.35"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vpro-maven2-group"
        NEXUS_LOGIN = "nexuslogin"
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests clean install'
            }
        }
    }
}