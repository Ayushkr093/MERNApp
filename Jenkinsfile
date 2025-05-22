pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' 
        FRONTEND_IMAGE = 'mern-frontend'
        BACKEND_IMAGE = 'mern-backend'
        DOCKER_NETWORK = 'demo'
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
                sh 'docker run --name=frontend --network=demo -d -p 5173:5173 mern-frontend'
            }
        }

        stage('Run MongoDB Container') {
            steps {
                sh 'docker rm -f mongodb || true'
                sh 'docker run --network=demo --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest'
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
                sh 'docker run --name=backend --network=demo -d -p 5050:5050 mern-backend'
            }
        }

        stage('Verify Frontend is Running') {
            steps {
                echo 'Check http://localhost:5173 in your browser'
                // Optional: use curl for automated check
                sh 'curl -s http://localhost:5173 || echo "Frontend may not be ready yet."'
            }
        }

        stage('Docker Compose (Optional)') {
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
            echo 'Pipeline failed ❌'
        }
        success {
            echo 'MERN stack successfully deployed with Docker 🐳'
        }
    }
}
