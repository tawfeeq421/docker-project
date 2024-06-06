pipeline {
    agent any
    tools {
        maven "maven"
        jdk "jdk"
        // Add SonarQube scanner tool definition here if not already defined
        // sonarScanner "sonar-scanner"
    }
    stages {
        stage('Fetch code') {
            steps {
                git url: 'https://github.com/tawfeeq421/docker-project.git'
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
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Checkstyle Analysis') {
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
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectName=javaapp \
                        -Dsonar.projectKey=javaapp \
                        -Dsonar.java.binaries=target/classes'''
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    // Change directory for the build command
                    dir('Docker-files') {
                        // Use withDockerRegistry to authenticate DockerHub
                        withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                            sh 'docker build -t app .'
                            sh 'docker tag app tawfeeq421/java11:task'
                            sh 'docker push tawfeeq421/java11:task'
                        }
                    }
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name app -p 3000:8080 tawfeeq421/java11:task'
            }
        }
    }
}
