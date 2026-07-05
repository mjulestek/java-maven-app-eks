#!/usr/bin/env groovy

pipeline {
    agent any

    tools {
        maven 'maven-3.9'
    }

    environment {
        DOCKER_REPO = 'mujuules01/demo-app'
        GITHUB_REPO = 'YOUR_REPO_NAME'
        GITHUB_REPO_OWNER = 'mjulestek'
        GIT_BRANCH_TO_PUSH = 'deploy-to-k8s'
        AWS_REGION = 'eu-central-1'
        EKS_CLUSTER_NAME = 'jennifer-demo-cluster'
    }

    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing the version...'

                    sh 'mvn build-helper:parse-version versions:set -DnewVersion=\\${parsedVersion.majorVersion}.\\${parsedVersion.minorVersion}.\\${parsedVersion.nextIncrementalVersion} versions:commit'

                    env.IMAGE_TAG = sh(
                        script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout',
                        returnStdout: true
                    ).trim()

                    env.DOCKER_IMAGE = "${DOCKER_REPO}:${IMAGE_TAG}"

                    echo "New app version is: ${IMAGE_TAG}"
                    echo "Docker image will be: ${DOCKER_IMAGE}"
                }
            }
        }

        stage('build app') {
            steps {
                script {
                    echo 'building the application...'
                    sh 'mvn package'
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'

                    withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                        sh 'docker build -t $DOCKER_IMAGE .'
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('deploy to EKS') {
            steps {
                script {
                    echo 'deploying application to EKS...'

                    withCredentials([
                        string(credentialsId: 'jenkins-aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'),
                        string(credentialsId: 'jenkins-aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')
                    ]) {
                        sh 'export AWS_PAGER="" && aws sts get-caller-identity'
                        sh 'export AWS_PAGER="" && aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER_NAME'
                        sh 'kubectl get nodes'
                        sh 'kubectl apply -f k8s/'
                        sh 'kubectl set image deployment/java-maven-app java-maven-app=$DOCKER_IMAGE'
                        sh 'kubectl rollout status deployment/java-maven-app'
                    }
                }
            }
        }

        stage('commit version update to git repo') {
            steps {
                script {
                    echo 'commit version update to git repo...'

                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GITHUB_TOKEN', usernameVariable: 'GITHUB_USER')]) {
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh 'git status'
                        sh 'git branch'

                        sh 'git remote set-url origin https://$GITHUB_USER:$GITHUB_TOKEN@github.com/$GITHUB_REPO_OWNER/$GITHUB_REPO.git'
                        sh 'git remote -v'

                        sh 'git fetch origin $GIT_BRANCH_TO_PUSH'
                        sh 'git checkout -B $GIT_BRANCH_TO_PUSH origin/$GIT_BRANCH_TO_PUSH'

                        sh 'git add .'
                        sh 'git commit -m "jenkins increment version [skip ci]" || true'
                        sh 'git pull --rebase origin $GIT_BRANCH_TO_PUSH'
                        sh 'git push origin HEAD:$GIT_BRANCH_TO_PUSH'
                    }
                }
            }
        }
    }
}