pipeline {
    /* insert Declarative Pipeline here */
    agent any
    tools {
        maven 'maven3'
        }
    options {
        buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '5', numToKeepStr: '3')
        }


    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Upload War To Nexus') {
            steps {
                script{
                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "maven-app1-snapshots" : "maven-app1-releases"
                    nexusArtifactUploader artifacts: [[
                        artifactId: 'maven-app1', 
                        classifier: '', 
                        file: "target/maven-app1-${mavenPom.version}.war", 
                        type: 'war']], 
                        credentialsId: 'nexus-credentials', 
                        groupId: 'in.javahome', 
                        nexusUrl: 'nexus:8081', 
                        nexusVersion: 'nexus3', 
                        protocol: 'http', 
                        repository: nexusRepoName, 
                        version: "${mavenPom.version}"

                } 
            }
        }
    }
}