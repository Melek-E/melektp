pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *') // Corrected cron expression syntax
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub') // Correct DockerHub credentials reference
        IMAGE_NAME_SERVER = 'melek16/mern-server' // Updated with your Docker Hub username
        IMAGE_NAME_CLIENT = 'melek16/mern-client' // Updated with your Docker Hub username
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/Melek-E/melektp.git', // Corrected GitHub URL
                    credentialsId: 'Github_ssh' // Corrected to use GitHub credentials
            }
        }
        
        stage('Build Server Image') {
            steps {
                dir('server') { // `server` is now directly inside `mern-app`
                    script {
                        dockerImageServer = docker.build("${IMAGE_NAME_SERVER}")
                    }
                }
            }
        }
        
        stage('Build Client Image') {
            steps {
                dir('client') { // `client` is at the root level inside `mern-app`
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
