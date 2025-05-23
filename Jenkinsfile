pipeline {
    agent any  // Use any agent to run the pipeline

    environment {
        // Docker Hub credentials and image names
        DOCKER_REGISTRY = 'ayushkr08'
        FRONTEND_IMAGE = 'mernapp-frontend'
        GIT_BRANCH = 'main' // You can change this to the branch you want to monitor
    }

    stages {
        // Git Clone the repository
        stage('Clone Git Repository') {
            steps {
                script {
                    echo 'Cloning repository...'
                    // Clone the repository from GitHub
                    sh 'git clone https://github.com/Ayushkr093/MERNApp.git'
                    dir('MERNApp') {
                        sh 'git checkout ${GIT_BRANCH}'  // Checkout the specific branch
                    }
                }
            }
        }

        // Build the frontend Docker image
        stage('Build Frontend Image') {
            steps {
                script {
                    echo 'Building frontend image...'
                    docker.build("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}", './MERNApp/mern/frontend')
                }
            }
        }

        // Login to Docker Hub
        stage('Login to Docker Hub') {
            steps {
                script {
                    echo 'Logging in to Docker Hub...'
                    docker.withRegistry("https://index.docker.io/v1/", 'dockerhub-credentials') {
                        // Credentials will be automatically used here
                    }
                }
            }
        }

        // Push Docker frontend image to Docker Hub
        stage('Push Frontend Docker Image to Docker Hub') {
            steps {
                script {
                    echo 'Pushing frontend image to Docker Hub...'
                    docker.withRegistry("https://index.docker.io/v1/", 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()
                    }
                }
            }
        }
    }

    post {
        always {
            // Cleanup any lingering Docker resources
            echo 'Cleaning up any remaining Docker resources...'
            sh 'docker system prune -f'
        }
    }
}
