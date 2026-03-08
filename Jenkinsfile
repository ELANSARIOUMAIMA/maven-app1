pipeline {
    /* insert Declarative Pipeline here */
    agent any
    tools {
        maven 'maven3'
        }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload War To Nexus') {
            steps {
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'maven-app1', 
                        classifier: '', 
                        file: 'target/maven-app1-1.0.0.war', 
                        type: 'war']], 
                        credentialsId: 'nexus-credentials', 
                        groupId: 'in.javahome', 
                        nexusUrl: 'nexus:8081', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: 'maven-app1-releases', 
                        version: '1.0.0'
            }
        }
    }
}