pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Cron expression to check every 5 minutes
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Docker Hub credentials reference
        IMAGE_NAME_SERVER = 'melek16/mern-server' // Updated with your Docker Hub username
        IMAGE_NAME_CLIENT = 'melek16/mern-client' // Updated with your Docker Hub username
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',  // Changed to 'main' from 'master'
                    url: 'git@github.com:Melek-E/melektp.git', // GitHub SSH URL
                    credentialsId: '0e6e9e06-926d-4608-acb6-4505cbfa1da5' // Using your GitHub SSH credentialsId
            }
        }
        
        stage('Build Server Image') {
            steps {
                dir('mern-app/server') {
                    sh 'pwd' // Debugging: print working directory
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        
        stage('Build Client Image') {
            steps {
                dir('mern-app/client') {
                    sh 'pwd' // Debugging: print working directory
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
