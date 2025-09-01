pipeline {
    agent any

    tools {
        // Matches tool name configured in Jenkins → Global Tool Configuration
        maven "MVN_HOME"
    }

    environment {
        // Nexus details
        NEXUS_VERSION       = "nexus3"
        NEXUS_PROTOCOL      = "http"
        NEXUS_URL           = "18.221.189.193:8081"   // Replace with your Nexus EC2 public/private IP
        NEXUS_REPOSITORY    = "maven-releases"       // Use the correct repo (not sonarqube!)
        NEXUS_CREDENTIAL_ID = "nexus_keygen"

        // SonarQube Scanner (configured in Jenkins → Tools)
        SCANNER_HOME = tool 'sonar_scanner'
    }

    stages {
        stage("Clone Code") {
            steps {
                git 'https://github.com/betawins/sabear_simplecutomerapp.git'
            }
        }

        stage("Build with Maven") {
            steps {
                sh 'mvn -Dmaven.test.failure.ignore=true clean install'
            }
        }

        stage("SonarQube Analysis") {
    steps {
        withSonarQubeEnv('sonarqube_server') {
            sh '''
                $SCANNER_HOME/bin/sonar-scanner \
                  -Dsonar.projectKey=Ncodeit \
                  -Dsonar.projectName=Ncodeit \
                  -Dsonar.projectVersion=2.0 \
                  -Dsonar.sources=$WORKSPACE/src \
                  -Dsonar.java.binaries=$WORKSPACE/target/SimpleCustomerApp-28-SNAPSHOT/WEB-INF/classes \
                  -Dsonar.junit.reportsPath=$WORKSPACE/target/surefire-reports \
                  -Dsonar.jacoco.reportPath=$WORKSPACE/target/jacoco.exec
            '''
        }
    }
}


        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    def pom = readMavenPom file: "pom.xml"
                    def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")

                    if (filesByGlob.length == 0) {
                        error "*** No artifact found in target directory"
                    }

                    def artifactPath = filesByGlob[0].path
                    echo "*** Uploading artifact: ${pom.groupId}:${pom.artifactId}:${pom.version} (${artifactPath})"

                    nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                            [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                        ]
                    )
                }
            }
        }
    }
}
