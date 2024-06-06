pipeline {
    agent any
    tools {
	    maven "maven"
	    jdk "jdk"
	}
    stages{
        stage('Fetch code') {
          steps{
              git url:'https://github.com/tawfeeq421/docker-project.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar-scanner'
            }
            steps {
               withSonarQubeEnv('sonar-server') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }

         stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){
                       sh "cd Docker-files"
                       sh "docker build -t app ."
                       sh "docker tag app tawfeeq421/java11:task "
                       sh "docker push tawfeeq421/java11:task "
                    }
                }
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name app -p 3000:8080 tawfeeq421/java11:task'
            }
        }



    }
}
