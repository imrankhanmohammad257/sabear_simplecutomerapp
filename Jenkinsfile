node('') {
    // Define Maven tool (must match name in Global Tool Config)
    def mvnHome = tool name: 'MVN_HOME', type: 'maven'

    // Define servers/credentials
    def sonarqubeServer = 'MySonarQube'                // must match Configure System > SonarQube
    def nexusUrl = "18.216.151.197:8081"
    def nexusRepo = "maven-releases"                   // adjust if different
    def nexusCred = "nexus_keygen"                     // Jenkins credential ID for Nexus
    def tomcatServer = "http://<TOMCAT_SERVER>:8080/manager/text"
    def tomcatCred = "tomcat-manager-creds"            // Jenkins credential ID for Tomcat
    def slackChannel = "#devops-alerts"                // Slack channel configured in Jenkins

    try {
        stage('Git Clone') {
            checkout([$class: 'GitSCM',
                branches: [[name: '*/feature-1.1']],
                userRemoteConfigs: [[url: 'https://github.com/imrankhanmohammad257/sabear_simplecutomerapp.git']]
            ])
        }

        stage('SonarQube Analysis') {
            withSonarQubeEnv(sonarqubeServer) {
                sh "${mvnHome}/bin/mvn sonar:sonar"
            }
        }

        stage('Maven Build') {
            sh "${mvnHome}/bin/mvn clean package -DskipTests"
        }

        stage('Publish to Nexus') {
            // ⚠️ Comment out this block if Nexus isn’t ready
            def pom = readMavenPom file: "pom.xml"
            def filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
            if (filesByGlob.length == 0) {
                error "No artifact found in target/"
            }
            def artifactPath = filesByGlob[0].path

            nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: nexusUrl,
                groupId: pom.groupId,
                version: pom.version,
                repository: nexusRepo,
                credentialsId: nexusCred,
                artifacts: [
                    [artifactId: pom.artifactId, classifier: '', file: artifactPath, type: pom.packaging],
                    [artifactId: pom.artifactId, classifier: '', file: "pom.xml", type: "pom"]
                ]
            )
        }

        stage('Deploy on Tomcat') {
            withCredentials([usernamePassword(credentialsId: tomcatCred, usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                def warFile = "target/sabear_simplecutomerapp.war"
                sh """
                    curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file ${warFile} "${tomcatServer}/deploy?path=/sabear_simplecutomerapp&update=true"
                """
            }
        }

        stage('Slack Notification') {
            slackSend(channel: slackChannel, color: 'good', message: "✅ Build & Deploy successful for feature-1.1 branch")
        }

    } catch (err) {
        slackSend(channel: slackChannel, color: 'danger', message: "❌ Build failed for feature-1.1 branch: ${err}")
        throw err
    }
}
