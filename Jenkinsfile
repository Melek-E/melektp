pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Corrected cron expression syntax
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Docker Hub credentials reference
        IMAGE_NAME_SERVER = 'melek16/mern-server' // Updated with your Docker Hub username
        IMAGE_NAME_CLIENT = 'melek16/mern-client' // Updated with your Docker Hub username
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'git@github.com:Melek-E/melektp.git', // GitHub SSH URL
                    credentialsId: '0e6e9e06-926d-4608-acb6-4505cbfa1da5' // Using the provided GitHub SSH credentialsId
            }
        }
        
        stage('Build Server Image') {
            steps {
                dir('server') { // Assuming server is inside 'mern-app/server'
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        
        stage('Build Client Image') {
            steps {
                dir('client') { // Assuming client is at the root level
                    script {
                        dockerImageClient = docker.build("${IMAGE_NAME_CLIENT}")
                    }
                }
            }
        }
        
        stage('Scan Server Image') {
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                    aquasec/trivy:latest image --exit-code 0 \\
                    --severity LOW,MEDIUM,HIGH,CRITICAL \\
                    ${IMAGE_NAME_SERVER}
                    """
                }
            }
        }

        stage('Scan Client Image') {
            steps {
                script {
                    sh """
                    docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \\
                    aquasec/trivy:latest image --exit-code 0 \\
                    --severity LOW,MEDIUM,HIGH,CRITICAL \\
                    ${IMAGE_NAME_CLIENT}
                    """
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKERHUB_CREDENTIALS}") {
                        dockerImageServer.push()
                        dockerImageClient.push()
                    }
                }
            }
        }
    }
}
