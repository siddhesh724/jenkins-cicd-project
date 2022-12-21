def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    tools {
        jdk "OracleJDK8"
        maven "MAVEN_LOCAL"
    }
    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "573421063144.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
        vprofileRegistry = "https://573421063144.dkr.ecr.us-east-1.amazonaws.com"
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
        
        stage('Build Image') {
            steps{
                script{
                    dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./")
                }
            }
        }

        stage('Upload App image') {
            steps{
                script {

                   docker.withRegistry( vprofileRegistry, registryCredential ) {
                     dockerImage.push("$BUILD_NUMBER")
                     dockerImage.push('latest')
                  }
                }
            }
        }



    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
    
}