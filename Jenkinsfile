def gv

pipeline {
    agent any
    tools {
        maven 'maven3'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                }
            }
        }
        stage("build jar") {
            steps {
                script { 
                    sh 'mvn package'
                    echo "building jar"
                    gv.buildJar()
                }
            }
        }
        stage("build image") {
            steps {
                script { 
                    sh 'docker build -t myapp:1.0 .'
                    echo "building image"
                    gv.buildImage()
                }
            }
        }
        stage("deploy") {
            steps {
                script { 
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'passwd', usernameVariable: 'username')]) {
    // some block
}
                    echo "deploying"
                    gv.deployApp()
                }
            }
        }
    }   
}
