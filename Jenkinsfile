pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main'  
        FRONTEND_IMAGE = 'ayushkr093/mernapp-frontend'
        BACKEND_IMAGE = 'ayushkr093/mernapp-backend'
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

        
        stage('Login to Docker Hub') {
    steps {
        echo "🔐 Logging into Docker Hub..."

        // Using Jenkins credentials to securely access Docker Hub credentials
        withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            // Use the Personal Access Token (DOCKER_PASSWORD) for login
            sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
        }
    }
}


        stage('Push Docker Images') {
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
    }

    post {
        success {
            echo '✅ Build and push to Docker Hub succeeded!'
        }
        failure {
            echo '❌ Build or push failed.'
        }
    }
}
