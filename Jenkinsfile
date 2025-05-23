pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'ayushkr08'
        FRONTEND_IMAGE = 'mernapp-frontend'
        BACKEND_IMAGE = 'mernapp-backend'
        GIT_BRANCH = 'main'
        GIT_REPO_URL = 'https://github.com/Ayushkr093/MERNApp.git'
    }

    stages {
        stage('Clone Git Repository') {
            steps {
                script {
                    echo '🔄 Cloning repository...'
                    sh 'rm -rf MERNApp'
                    sh "git clone ${GIT_REPO_URL}"
                    dir('MERNApp') {
                        sh "git checkout ${GIT_BRANCH}"
                    }
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo '🏗️ Building frontend image...'
                    docker.build("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}", './MERNApp/mern/frontend')

                    echo '🏗️ Building backend image...'
                    docker.build("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}", './MERNApp/mern/backend')
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    echo '🔐 Logging in to Docker Hub...'
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDENTIALS', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin'
                    }
                }
            }
        }

        stage('Push Docker Images to Docker Hub') {
            steps {
                script {
                    echo '📤 Pushing frontend image...'
                    docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()

                    echo '📤 Pushing backend image...'
                    docker.image("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}").push()
                }
            }
        }

        stage('Run with Docker Compose') {
            steps {
                script {
                    echo '🚀 Running app with Docker Compose...'
                    dir('MERNApp') {
                        sh 'docker compose up -d'
                    }
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up unused Docker resources...'
            sh 'docker system prune -f --volumes || true'
        }
    }
}
