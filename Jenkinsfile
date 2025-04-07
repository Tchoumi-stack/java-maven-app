pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    } 
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/Tchoumi-stack/java-maven-app.git'                    
            }
        }
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('file system scan') {
            steps {
                sh 'trivy fs .'
            }
        }    
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=java-maven-app -Dsonar.projectkey=java-maven-app \
                          -Dsonar.java.binaries=. '''
                }
            }
        }
      stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        stage('Build Artifact') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Publish to Nexus') {
            steps {
                script {
                    
                } 
            }
        }
        stage('Build & Tag Image') {
            steps {
                script {

                      sh 'docker build -t minelva298/java-maven-app:1.0 .'
                
                }    
            }
        }
        stage('Docker Scan Image') {
            steps {
                sh 'trivy image minelva298/java-maven-app:1.0 .'
            }
        }
        stage('Docker push image') {
            steps {
                script {
                    
                 sh 'docker push minelva298/java-maven-app:1.0'
            }
        }
          stage('Deploy To Kubernetes') {
            steps {
                script {
                sh 'kubectl apply -f deployment.yaml'
            }
        }
         stage('Verify The Deployment') {
            steps {
                script {
                    sh 'kubectl get pods -n webapp'
                    sh 'kubectl get svc -n webapp'
                sh 'trivy fs .'
            }
        }
             
         
