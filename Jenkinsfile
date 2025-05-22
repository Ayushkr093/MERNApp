pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' 
        FRONTEND_IMAGE = 'mern-frontend'
        BACKEND_IMAGE = 'mern-backend'
        DOCKER_NETWORK = 'mern-network'
        FRONTEND_CONTAINER = 'frontend'
        BACKEND_CONTAINER = 'backend'
        MONGO_CONTAINER = 'mongodb'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "${env.GIT_URL}", branch: 'main'
            }
        }

        stage('Create Docker Network') {
            steps {
                script {
                    def networkExists = sh(script: "docker network ls --filter name=^${DOCKER_NETWORK}\$ --format '{{.Name}}'", returnStdout: true).trim()
                    if (networkExists == '') {
                        sh "docker network create ${DOCKER_NETWORK}"
                    } else {
                        echo "Network '${DOCKER_NETWORK}' already exists"
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('mern/frontend') {
                    sh "docker build -t ${FRONTEND_IMAGE} ."
                }
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh "docker rm -f ${FRONTEND_CONTAINER} || true"
                sh "docker run --name=${FRONTEND_CONTAINER} --network=${DOCKER_NETWORK} -d -p 5173:5173 ${FRONTEND_IMAGE}"
            }
        }

        stage('Run MongoDB Container') {
            steps {
                sh "docker rm -f ${MONGO_CONTAINER} || true"
                sh "docker run --network=${DOCKER_NETWORK} --name=${MONGO_CONTAINER} -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest"
            }
        }

        stage('Build Backend') {
            steps {
                dir('mern/backend') {
                    sh "docker build -t ${BACKEND_IMAGE} ."
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh "docker rm -f ${BACKEND_CONTAINER} || true"
                sh "docker run --name=${BACKEND_CONTAINER} --network=${DOCKER_NETWORK} -d -p 5050:5050 ${BACKEND_IMAGE}"
            }
        }

        stage('Verify Frontend is Running') {
            steps {
                echo 'Check http://localhost:5173 in your browser'
                sh 'curl -s http://localhost:5173 || echo "Frontend may not be ready yet."'
            }
        }

        stage('Docker Compose') {
            when {
                expression { fileExists('docker-compose.yml') }
            }
            steps {
                sh 'docker compose up -d'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed ❌ Cleaning up Docker resources...'
            script {
                sh "docker rm -f ${FRONTEND_CONTAINER} ${BACKEND_CONTAINER} ${MONGO_CONTAINER} || true"
                sh "docker network rm ${DOCKER_NETWORK} || true"
            }
        }

        success {
            echo '✅ MERN stack successfully deployed with Docker 🐳'
        }
    }
}
