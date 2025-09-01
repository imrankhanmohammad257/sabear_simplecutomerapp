node('master') {
    def mvnHome = tool name: 'MVN_HOME', type: 'maven'

    // Environment variables
    def sonarqubeServer = 'MySonarQube'        // configure this in Jenkins
    def nexusUrl = "18.216.151.197:8081"
    def nexusRepo = "maven-releases"           // adjust to your Nexus repo
    def nexusCred = "nexus_keygen"             // Jenkins credential ID
    def tomcatServer = "http://<TOMCAT_SERVER>:8080/manager/text"
    def tomcatUser = "admin"
    def tomcatPass = "password"
    def slackChannel = "#devops-alerts"

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
            // ⚠️ disable this block if Nexus not ready
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
            def warFile = "target/sabear_simplecutomerapp.war"
            sh """
                curl -u ${tomcatUser}:${tomcatPass} --upload-file ${warFile} "${tomcatServer}/deploy?path=/sabear_simplecutomerapp&update=true"
            """
        }

        stage('Slack Notification') {
            slackSend(channel: slackChannel, color: 'good', message: "Build & Deploy successful for feature-1.1 ✅")
        }

    } catch (err) {
        slackSend(channel: slackChannel, color: 'danger', message: "Build failed ❌: ${err}")
        throw err
    }
}
