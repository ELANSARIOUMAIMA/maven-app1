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
    }
}