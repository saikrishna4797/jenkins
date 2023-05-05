pipeline {
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "Maven3.9"
    }

    stages {
        stage('checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/keyspaceits/javawebapp.git']])
            }
        }
        stage('build') {
            steps {
                sh 'mvn clean install -f pom.xml'
            }
        
        }
        stage('build-notify') {
            steps {
                slackSend channel: 'opsteam', message: 'build success', teamDomain: 'creativeworks-corp', tokenCredentialId: 'slack'
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
                deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://34.234.40.136:8080')], contextPath: null, war: '**/*.war'
            }
        }
        stage('Deploy-notify') {
            steps {
                slackSend channel: 'opsteam', message: 'Deployment To QA Success', teamDomain: 'creativeworks-corp', tokenCredentialId: 'slack'
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
                deploy adapters: [tomcat9(credentialsId: 'tomcat', path: '', url: 'http://34.234.40.136:8080')], contextPath: null, war: '**/*.war'
            }
        }
    }
}
