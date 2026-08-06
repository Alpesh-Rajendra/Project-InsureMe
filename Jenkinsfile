pipeline {
    agent any 

    tools {
        maven 'maven'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "project-insure-me-build-artifacts-b31"
        REGION = "us-east-2"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
    }

    stages {

        stage('code-pull') {
            steps {
                git branch: 'main', url: 'https://github.com/Alpesh-Rajendra/Project-InsureMe.git'
            }
        }

        stage('code-build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
         stage('code-push'){
            steps{
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                   sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
                 }
            }
        }

        stage("code-test-analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=InsureMe \
                        -Dsonar.projectName=InsureMe \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }

        stage("code-test-quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
     stage('docker-image'){
            steps{
                sh 'docker build -t kaali28/insure-b31 .'
                
            }
        }
        
        stage('image-push'){
            steps {
       	       withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
            	sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                sh 'docker push kaali28/insure-b31'
               }
            }
        } 
        
        stage('code-deploy'){
            steps{
                sh 'docker run -itd --name insure-me -p 8089:8081 kaali28/insure-b31'
            }
        }

    }
}
