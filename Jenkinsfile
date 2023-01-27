#! /usr/bin/env groovy

library identifier: 'jenkins-shared-library@main', retriever: modernSCM(
    [
        $class: 'GitSCMSource',
        remote: 'git@github.com:daniellehopedev/jenkins-shared-library.git',
        credentialsId: 'github-ssh-credentials'
    ]
)

pipeline {
    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = 'danielleh/my-repo:maven-app-4.0'
    }

    stages {
        stage('build app') {
            steps {
                script {
                    echo 'building application jar...'
                    buildJar()
                }
            }
        }

        stage('build image') {
            steps {
                script {
                    echo 'building docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker run -d -p 8080:8080 ${IMAGE_NAME}"
                    sshagent(['ec2-ssh-credentials']) {
                        // -o StrictHostKeyChecking=no, suppresses popup when connecting to the ec2 server
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@18.191.57.52 ${dockerCmd}"
                    }
                }
            }
        }
    }   
}