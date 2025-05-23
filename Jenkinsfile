pipeline {
    agent any
    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main'

        DOCKER_HUB_REPO = 'ayushkr093/mernapp'
        DOCKER_HUB_USERNAME = credentials('ayushkr08') // Injected securely
        DOCKER_HUB_PASSWORD = credentials('dckr_pat_pS0HgaOllLoES4QPxSIe0h7GGI8') // Injected securely

        FRONTEND_IMAGE = "${DOCKER_HUB_REPO}-frontend:latest"
        BACKEND_IMAGE = "${DOCKER_HUB_REPO}-backend:latest"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "${env.GIT_URL}", branch: "${env.GIT_BRANCH}"
            }
        }

        stage('Build Images') {
            parallel {
                stage('Frontend') {
                    steps {
                        dir('mern/frontend') {
                            sh "docker build -t ${env.FRONTEND_IMAGE} ."
                        }
                    }
                }
                stage('Backend') {
                    steps {
                        dir('mern/backend') {
                            sh "docker build -t ${env.BACKEND_IMAGE} ."
                        }
                    }
                }
            }
        }

        stage('Login & Push') {
            steps {
                sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                sh "docker push ${env.FRONTEND_IMAGE}"
                sh "docker push ${env.BACKEND_IMAGE}"
            }
        }
    }

    post {
        failure {
            echo 'Build failed. No images pushed.'
        }
    }
}
