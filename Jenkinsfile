pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'ayushkr08'
        FRONTEND_IMAGE = 'mernapp-frontend'
        BACKEND_IMAGE  = 'mernapp-backend'
        MONGODB_IMAGE  = 'mernapp-mongodb'
        GIT_BRANCH     = 'main'
        GIT_REPO_URL   = 'https://github.com/Ayushkr093/MERNApp.git'
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

                    echo '🏗️ Building MongoDB image...'
                    docker.build("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}", './MERNApp/mern/mongodb')
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

        stage('Push Docker Images') {
            steps {
                script {
                    echo '📤 Pushing frontend image...'
                    docker.image("${DOCKER_REGISTRY}/${FRONTEND_IMAGE}:${GIT_BRANCH}").push()

                    echo '📤 Pushing backend image...'
                    docker.image("${DOCKER_REGISTRY}/${BACKEND_IMAGE}:${GIT_BRANCH}").push()

                    echo '📤 Pushing MongoDB image...'
                    docker.image("${DOCKER_REGISTRY}/${MONGODB_IMAGE}:${GIT_BRANCH}").push()
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    echo '🚀 Deploying using Docker Compose...'
                    dir('MERNApp/mern') {
                        sh 'docker-compose down || true'
                        sh 'docker-compose up -d'
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
