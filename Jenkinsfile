pipeline {
    agent any
    stages {
        stage('Checkout SCM') {
            steps {
                git url: 'https://github.com/imrankhanmohammad257/sabear_simplecutomerapp.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                tool name: 'Maven-3.8.4', type: 'maven'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Deploy to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh 'mvn clean deploy -DskipTests -Dnexus.username=$NEXUS_USER -Dnexus.password=$NEXUS_PASS --settings /var/lib/jenkins/.m2/settings.xml'
                }
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                         -T target/hiring.war \
                         http://54.145.142.96:8080/manager/text/deploy?path=/hiring&update=true
                    '''
                }
            }
        }

        stage('Slack Notification') {
    steps {
        slackSend(
            channel: '#jenkins-integration',
            color: 'good',
            message: "Hi Team, Jenkins Challenge Declarative pipeline job for *hiring-app* has finished successfully! ✅\nDeployed by: Imran Khan"
        )
    }
}

        
    }
    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
