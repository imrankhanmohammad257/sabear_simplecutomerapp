node {
    stage('Git Clone') {
        git branch: 'feature-1.1',
            url: 'https://github.com/imrankhanmohammad257/sabear_simplecutomerapp.git'
    }

    stage('SonarQube Analysis') {
        withSonarQubeEnv('SonarQube') {
            sh 'mvn sonar:sonar'
        }
    }

    stage('Maven Compilation') {
        def mvnHome = tool name: 'Maven-3.8.4', type: 'maven'
        sh "${mvnHome}/bin/mvn clean compile"
    }

    stage('Deploy to Nexus') {
        withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
            sh '''
            mvn clean deploy -DskipTests --settings /var/lib/jenkins/.m2/settings.xml
            '''
        }
    }

    stage('Slack Notification') {
        slackSend(
            channel: '#jenkins-integration',
            color: 'good',
            message: "Jenkins Scripted Pipeline job for *sabear_simplecutomerapp (feature-1.1)* finished successfully! ✅"
        )
    }


stage('Deploy to Tomcat') {
    withCredentials([usernamePassword(credentialsId: 'tomcat-credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
        sh '''
        WAR_FILE=$(ls target/*.war | head -n 1)

        echo "Undeploying old app..."
        curl -s -o /dev/null -w "%{http_code}" -u $TOMCAT_USER:$TOMCAT_PASS \
            "http://54.87.222.232:8080/manager/text/undeploy?path=/customerapp"

        echo "Deploying new WAR: $WAR_FILE"
        curl -s -o /dev/null -w "%{http_code}" -u $TOMCAT_USER:$TOMCAT_PASS \
            -T $WAR_FILE \
            "http://54.87.222.232:8080/manager/text/deploy?path=/customerapp&update=true"

        echo "Deployment completed."
        '''
    }
}



    
}
