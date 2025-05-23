pipeline {
    agent any

    environment {
        // Docker Hub Credentials (Use Jenkins' Docker Credentials Store or Secrets)
        DOCKER_CREDENTIALS = credentials('docker-hub-credentials') // Replace with your Jenkins Docker Hub credentials ID
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_USER = "${DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD = "${DOCKER_CREDENTIALS_PSW}"
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main' // Trigger on changes to 'develop' branch
        FRONTEND_IMAGE = 'mernapp_frontend'
        BACKEND_IMAGE = 'mernapp_backend'
        MONGODB_IMAGE = 'mernapp_mongodb'
    }

    triggers {
        // Trigger the pipeline on a push to the develop branch
        githubPush() // Ensure that GitHub webhook is set up to trigger on push to develop branch
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from the Git repository
                git url: "${env.GIT_URL}", branch: "${env.GIT_BRANCH}"
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    echo 'Building frontend image...'
                    docker.build("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}", './mern/frontend') // Build from MERNApp/mern/frontend directory
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                script {
                    echo 'Building backend image...'
                    docker.build("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}", './mern/backend') // Build from MERNApp/mern/backend directory
                }
            }
        }

        stage('Build MongoDB Image') {
            steps {
                script {
                    echo 'Building MongoDB image...'
                    docker.build("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}", './mern/mongodb') // Build from MERNApp/mern/mongodb directory (if custom setup is needed)
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    echo 'Logging into Docker Hub...'
                    // Login to Docker Hub using Jenkins credentials
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
                        echo 'Successfully logged in to Docker Hub.'
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    echo 'Pushing frontend image to Docker Hub...'
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
                        docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()
                    }

                    echo 'Pushing backend image to Docker Hub...'
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
                        docker.image("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}").push()
                    }

                    echo 'Pushing MongoDB image to Docker Hub...'
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS}") {
                        docker.image("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}").push()
                    }
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    echo 'Running Docker Compose to start services...'
                    // Use docker-compose to start up the services using the newly pushed images
                    sh 'docker-compose -f MERNApp/docker-compose.yml up -d' // This should be set up to use the images from Docker Hub
                }
            }
        }
    }

    post {
        success {
            echo 'Build and deployment completed successfully!'
        }
        failure {
            echo 'Build or deployment failed. Performing cleanup...'
            // Clean up resources in case of failure
            sh 'docker-compose down --volumes --remove-orphans'
        }
    }
}
