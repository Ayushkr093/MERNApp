pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' 
        FRONTEND_IMAGE = 'mern-frontend'
        BACKEND_IMAGE = 'mern-backend'
        DOCKER_NETWORK = 'mern-network'
        FRONTEND_PORT = '5173'  // Port to be checked
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
                    def networkExists = sh(script: '''docker network ls --filter name=^${DOCKER_NETWORK}\\$ --format '{{.Name}}' ''', returnStdout: true).trim()
                    if (networkExists == '') {
                        sh "docker network create ${DOCKER_NETWORK}"
                    } else {
                        echo "Network '${DOCKER_NETWORK}' already exists"
                    }
                }
            }
        }

        stage('Clear Port if in Use') {
            steps {
                script {
                    // Check if the port is in use
                    def portInUse = sh(script: "lsof -i :${env.FRONTEND_PORT} || true", returnStatus: true)

                    if (portInUse == 0) {
                        // Port is in use, find the container using it
                        echo "Port ${env.FRONTEND_PORT} is in use. Stopping and removing the container using it..."
                        
                        // Find the container using the port and stop it
                        def containerId = sh(script: "lsof -t -i :${env.FRONTEND_PORT} || true", returnStdout: true).trim()
                        
                        if (containerId) {
                            sh "docker rm -f ${containerId} || true"
                            echo "Container using port ${env.FRONTEND_PORT} stopped and removed."
                        }
                    } else {
                        echo "Port ${env.FRONTEND_PORT} is not in use. Proceeding..."
                    }
                }
            }
        }

        stage('Build Frontend') {
            steps {
                dir('mern/frontend') {
                    sh 'docker build -t mern-frontend .'
                }
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh 'docker rm -f frontend || true'
                sh 'docker run --name=frontend --network=mern-network -d -p 5173:5173 mern-frontend'
            }
        }

        stage('Run MongoDB Container') {
            steps {
                sh 'docker rm -f mongodb || true'
                sh 'docker run --network=mern-network --name=mongodb -d -p 27017:27017 -v /var/lib/jenkins/opt/data:/data/db mongo:latest'
            }
        }

        stage('Build Backend') {
            steps {
                dir('mern/backend') {
                    sh 'docker build -t mern-backend .'
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh 'docker rm -f backend || true'
                sh 'docker run --name=backend --network=mern-network -d -p 5050:5050 mern-backend'
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
                sh 'docker rm -f frontend backend mongodb || true'
                sh "docker network rm ${DOCKER_NETWORK} || true"
            }
        }
        success {
            echo 'MERN stack successfully deployed with Docker 🐳'
        }
    }
}
