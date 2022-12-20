pipeline {
    agent any
    tools {
        jdk "OracleJDK8"
        maven "MAVEN_LOCAL"
    }
    stages {
        stage ('fetch the') {
            steps{
                git branch: 'vp-rem',url: 'https://github.com/devopshydclub/vprofile-project.git'
            }
        }
        stage ('Build') {
            steps{
                sh 'mvn clean install -DskipTest'
            }
        }
    }
}