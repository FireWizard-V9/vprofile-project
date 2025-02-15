pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"  // Ensure JDK 17 is used for build and tests
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.27.59'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = tool 'sonarscanner'
    }

    stages {
        stage('Verify Java Version') {
            steps {
                sh 'java -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        // ✅ Switch to JDK 11 for Sonar Analysis
        stage('Sonar Analysis') {
            steps {
                script {
                    echo "Switching to JDK 11 for Sonar Analysis"
                    sh '''
                    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
                    export PATH=$JAVA_HOME/bin:$PATH
                    java -version  # Verify Java version is switched
                    '''
                }
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${SONARSCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportsPath=target/surefire-reports/ \
                        -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
    }
}
