pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven"
    }

    stages {
        stage('checkout') {
            steps {
              checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/prasadrendla/jenkins.git']])
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean install -f pom.xml'
            }
        
        }
        stage('build-notify') {
            steps {
               slackSend channel: 'devops', message: 'success', tokenCredentialId: 'slack-note'
            }
        }
        stage('CodeQuality') {
            steps {
            withSonarQubeEnv('SonarQube') {
            sh 'mvn clean install -f pom.xml sonar:sonar'
            }
            }
        }
        stage('Nexus Upload'){
            steps {
            nexusArtifactUploader artifacts: [[artifactId: 'CounterWebApp', classifier: '', file: '/var/lib/jenkins/workspace/test-sonar/target/CounterWebApp.war', type: 'WAR']], credentialsId: 'nexus', groupId: 'com.mkyong', nexusUrl: '52.3.36.8:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        stage('Deploy to QA') {
            steps {
              deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://54.226.53.34:8080/')], contextPath: null, war: '**/*'
            }
        }
        stage('Deploy-notify') {
            steps {
               slackSend channel: 'devops', message: 'success', tokenCredentialId: 'slack-note'
            }
        }
        stage('Deploy to Prod Approve') {
            steps {
            echo "Taking approval from Manager"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to Proceed to Production?', submitter: 'admin'
                }
            }
        }
        stage('Deploy to Prod') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://54.226.53.34:8080/')], contextPath: null, war: '**/*'
            }
        }
    }
}
