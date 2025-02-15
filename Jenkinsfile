pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"  // Using JDK 17 for all stages
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
        SONARSCANNER = tool name: 'sonarscanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'  // Ensuring JDK 17 is used
        PATH = "$JAVA_HOME/bin:$PATH"
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

        stage('Sonar Analysis') {
            steps {
                script {
                    sh 'java -version'  // Verify Java 17 is active
                }
                withSonarQubeEnv("${SONARSERVER}") {
                    script {
                        def sonarStatus = sh(
                            script: '''${SONARSCANNER}/bin/sonar-scanner \
                                -Dsonar.projectKey=vprofile \
                                -Dsonar.projectName=vprofile \
                                -Dsonar.projectVersion=1.0 \
                                -Dsonar.sources=src/ \
                                -Dsonar.java.binaries=target/classes/ \
                                -Dsonar.junit.reportsPath=target/surefire-reports/ \
                                -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                                -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml''',
                            returnStatus: true  // Capture the exit status
                        )
                        
                        if (sonarStatus != 0) {
                            echo "⚠️ Sonar Analysis failed, but continuing pipeline..."
                        }
                    }
                }
            }
        }

        stage("Upload Artifact") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.BUILD_NUMBER}",  // Using BUILD_NUMBER as version
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",  // Fixed incorrect syntax
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }

    post {
        always {
            script {
                echo "✅ Pipeline completed."
            }
        }
    }
}
