pipeline {
    agent any
    tools {
        jdk "OracleJDK8"
        maven "MAVEN_LOCAL"
    }
    stages {
        stage ('fetch the') {
            steps{
                git branch: 'main',url: 'https://github.com/siddhesh724/jenkins-cicd-project.git'
            }
        }
        stage ('Build') {
            steps{
                sh 'mvn clean install -DskipTest'
            }
        }
        stage ('Test') {
            steps{
                sh 'mvn test'
            }
        }
        stage ('checkstyle analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage ('sonar analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('nexus artifact upload') {
            steps {
                nexusArtifactUploader(
                nexusVersion: 'nexus3',
                protocol: 'http',
                nexusUrl: '172.31.73.82:8081',
                groupId: 'QA',
                version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                repository: 'vprofile',
                credentialsId: 'nexus-login',
                artifacts: [
                  [artifactId: vproapp,
                  classifier: '',
                  file: 'target/vprofile-v2.war',
                  type: 'war']
                ]
                )
            }
        }
        
    }
}