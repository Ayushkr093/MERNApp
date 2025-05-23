pipeline {
    agent any  // Use any agent to run the pipeline

    environment {
        // Docker Hub credentials and image names
        DOCKER_REGISTRY = 'ayushkr08'
        FRONTEND_IMAGE = 'mernapp-frontend'
        BACKEND_IMAGE = 'mernapp-backend'
        MONGODB_IMAGE = 'mernapp-mongodb'
        GIT_BRANCH = 'main' // You can change this to the branch you want to monitor
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' // GitHub repository URL
    }

    stages {

        // Checkout the specified Git branch
        stage('Checkout Code') {
            steps {
                script {
                    echo "Cloning repository ${GIT_URL}..."
                    // Checkout the code from GitHub repository
                    git url: "${GIT_URL}", branch: "${GIT_BRANCH}"
                }
            }
        }

        // Build the frontend Docker image
        stage('Build Frontend Image') {
            steps {
                script {
                    echo 'Building frontend image...'
                    docker.build("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}", './mern/frontend')
                }
            }
        }

        // Build the backend Docker image
        stage('Build Backend Image') {
            steps {
                script {
                    echo 'Building backend image...'
                    docker.build("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}", './mern/backend')
                }
            }
        }

        // Build the MongoDB Docker image
        stage('Build MongoDB Image') {
            steps {
                script {
                    echo 'Building MongoDB image...'
                    docker.build("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}", './mern/mongodb')
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

        // Push Docker images to Docker Hub
        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    echo 'Pushing frontend image to Docker Hub...'
                    docker.withRegistry("https://index.docker.io/v1/", 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()
                    }

                    echo 'Pushing backend image to Docker Hub...'
                    docker.withRegistry("https://index.docker.io/v1/", 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}").push()
                    }

                    echo 'Pushing MongoDB image to Docker Hub...'
                    docker.withRegistry("https://index.docker.io/v1/", 'dockerhub-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}").push()
                    }
                }
            }
        }

        // Run Docker Compose to bring up containers
        stage('Run Docker Compose') {
            steps {
                script {
                    echo 'Running Docker Compose to start containers...'
                    sh 'docker-compose -f MERNApp/docker-compose.yml up -d'
                }
            }
        }

        // Clean up after build
        stage('Clean up') {
            steps {
                script {
                    echo 'Cleaning up Docker containers and networks...'
                    sh 'docker-compose -f MERNApp/docker-compose.yml down --volumes --remove-orphans'
                    sh 'docker network rm mern-network'
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
