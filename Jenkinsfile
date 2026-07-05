def gv

pipeline {   
    agent any
    tools {
        maven 'maven-3.9'
    }
    stages {
        stage("init") {
            steps {
                script {
                    echo 'Initializing pipeline...'
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                    echo 'Building JAR file...'

                }
            }
        }

        stage("build image") {
            steps {
                script {
                    echo 'build docker image...'
                }
            }
        }

        stage("deploy") {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins-aws-secret-access-id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
            }
            steps {
                script {
                    echo 'Deploying application...'
                    sh 'kubectl create deployment nginx-deployment --image=nginx'
                }
            }
        }               
    }
} 
