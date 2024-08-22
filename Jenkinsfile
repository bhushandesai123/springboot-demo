pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = 'sonar-scanner'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Fir3eye/springboot-demo.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html . "
            }
        }
        stage("SonarQube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner \
                                -Dsonar.projectName=springboot \
                                -Dsonar.projectKey=springboot \
                                -Dsonar.java.binaries=target'''
                    }
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage('Docker build and tag') {
            steps {
                sh "docker build -t fir3eye/new:latest ."
            }
        }
        stage('Trivy image Scan') {
            steps {
                sh "trivy image fir3eye/new:latest --format table -o image.html"
            }
        }
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker push fir3eye/new:latest"
                  }
                }
            }
        }
        stage('Deploy on Container') {
            steps {
                sh "docker run -d -p 8090:8080 fir3eye/new:latest"
            }
        }
    }
}
