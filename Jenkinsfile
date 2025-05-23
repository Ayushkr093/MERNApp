pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main'
        FRONTEND_IMAGE = 'ayushkr08/mernapp-frontend'
        BACKEND_IMAGE = 'ayushkr08/mernapp-backend'
        MONGO_IMAGE = 'mongo:latest'  
        BUILD_TAG = "v1-${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Checking out code from ${GIT_BRANCH}..."
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                echo "🏗️ Building Frontend Docker Image..."
                sh """
                    cd mern/frontend
                    docker build -t ${FRONTEND_IMAGE}:${BUILD_TAG} .
                    docker tag ${FRONTEND_IMAGE}:${BUILD_TAG} ${FRONTEND_IMAGE}:latest
                """
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                echo "🏗️ Building Backend Docker Image..."
                sh """
                    cd mern/backend
                    docker build -t ${BACKEND_IMAGE}:${BUILD_TAG} .
                    docker tag ${BACKEND_IMAGE}:${BUILD_TAG} ${BACKEND_IMAGE}:latest
                """
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                echo "🚀 Pushing Docker images to Docker Hub..."
                sh """
                    docker push ${FRONTEND_IMAGE}:${BUILD_TAG}
                    docker push ${FRONTEND_IMAGE}:latest
                    docker push ${BACKEND_IMAGE}:${BUILD_TAG}
                    docker push ${BACKEND_IMAGE}:latest
                """
            }
        }

        stage('Setup and Run Containers') {
            steps {
                echo "⚙️ Setting up network and running containers..."
                sh """
                    # Create the custom network
                    docker network create demo

                    # Run MongoDB container
                    docker run --network=demo --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest

                    # Run Frontend container
                    docker run --name=frontend --network=demo -d -p 5173:5173 ${FRONTEND_IMAGE}:${BUILD_TAG}

                    # Run Backend container
                    docker run --name=backend --network=demo -d -p 5050:5050 ${BACKEND_IMAGE}:${BUILD_TAG}

                    # Optionally verify if frontend is running by curling the frontend URL
                    curl http://localhost:5173
                """
            }
        }

        stage('Clean Up') {
            steps {
                echo "🧹 Cleaning up any resources if necessary..."
                sh """
                    # Clean up: Remove network after usage (optional)
                    docker network rm demo
                """
            }
        }
    }

    post {
        success {
            echo '✅ Build, push to Docker Hub, and run containers succeeded!'
        }
        failure {
            echo '❌ Build, push to Docker Hub, or run containers failed.'
        }
    }
}
