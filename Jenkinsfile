pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'ayushkr08'
        FRONTEND_IMAGE = 'mernapp-frontend'
        GIT_BRANCH = 'main'
        GIT_REPO_URL = 'https://github.com/Ayushkr093/MERNApp.git'
    }

    stages {
        stage('Clone Git Repository') {
            steps {
                script {
                    echo '🔄 Cloning repository...'
                    sh "git clone ${GIT_REPO_URL}"
                    dir('MERNApp') {
                        sh "git checkout ${GIT_BRANCH}"
                    }
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                script {
                    echo '🏗️ Building frontend Docker image...'
                    docker.build("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}", './MERNApp/mern/frontend')
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

        stage('Push Frontend Docker Image to Docker Hub') {
            steps {
                script {
                    echo '📤 Pushing frontend image to Docker Hub...'
                    docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()
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
