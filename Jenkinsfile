pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Build') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew build'
            }
        }
        stage('Git checkout') {
            steps {
                git credentialsId: 'git-cred', url: 'https://github.com/Tchoumi-stack/java-maven-app.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File System Scan') {
            steps {
                sh 'trivy fs .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=java-maven-app -Dsonar.projectKey=java-maven-app \
                    -Dsonar.java.binaries=.'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
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
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build & Tag Docker Image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub-cred', url: '') {
                    sh 'docker build -t minelva298/java-maven-app:1.0 .'
                }
            }
        }
        stage('Docker Scan Image') {
            steps {
                sh 'trivy image minelva298/java-maven-app:1.0'
            }
        }
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred', url: '') {
                        sh 'docker push minelva298/java-maven-app:1.0'
                    }
                }
            }
        }
        stage('Deploy To Kubernetes') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapp', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.47.56:6443') {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
        stage('Verify the Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapp', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.47.56:6443') {
                    sh 'kubectl get pods -n webapp'
                    sh 'kubectl get svc -n webapp'
                }
            }
        }
    }
}
            
