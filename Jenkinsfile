pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main'
        FRONTEND_IMAGE = 'ayushkr08/mernapp-frontend'
        BACKEND_IMAGE = 'ayushkr08/mernapp-backend'
        MONGO_IMAGE = 'ayushkr08/mernapp-mongo'  /
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

        stage('Build MongoDB Docker Image') {
            steps {
                echo "🏗️ Building MongoDB Docker Image..."
                sh """
                    docker pull mongo:latest
                    docker tag mongo:latest ${MONGO_IMAGE}:${BUILD_TAG}
                    docker tag mongo:latest ${MONGO_IMAGE}:latest
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo "🔐 Logging into Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                }
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
                    docker push ${MONGO_IMAGE}:${BUILD_TAG}
                    docker push ${MONGO_IMAGE}:latest
                """
            }
        }

        stage('Run Containers with Docker Compose') {
            steps {
                echo "⚙️ Running Docker containers with Docker Compose..."
                sh """
                    # Navigate to the project root where docker-compose.yml is located
                    cd ${WORKSPACE}
                    
                    # Run Docker Compose to bring up the services
                    docker-compose up -d
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
