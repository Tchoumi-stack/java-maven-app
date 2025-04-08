    
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
                  withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                            sh 'mvn deploy'
                   }
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                script {
                   withDockerRegistry(credentialsId: 'dockerhub-cred', toolName: 'docker') {
                            sh 'docker build -t minelva298/java-maven-app:1.0 .'
                   }
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
                  withDockerRegistry(credentialsId: 'dockerhub-cred', toolName: 'docker') {   
                  sh 'docker push minelva298/java-maven-app:1.0'
                  }
                }
            }
        }   
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapp', restrictKubeConfigAccess: false, serverUrl: ' https://172.31.47.56:6443') {
                          sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
        stage('verify the deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapp', restrictKubeConfigAccess: false, serverUrl: ' https://172.31.47.56:6443') {
                          sh 'kubectl get pods -n webapp'
                          sh 'kubectl get svc -n webapp'
                     }
            }
        }
                

                
         
        }            
