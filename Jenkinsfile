pipeline{
    agent any
    tools{
        maven "maven"
        jdk 'jdk'
        
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh 'docker build -t app .'
                        sh 'docker tag app tawfeeq421/java11:task'
                        sh 'docker push tawfeeq421/java11:task'
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