pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        DOCKER_NETWORK = 'mern-network'
        RETRY_LIMIT = 5
        RETRY_DELAY = 5 // Seconds
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "${env.GIT_URL}", branch: 'main'
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    echo 'Cleaning up existing containers on ports 27017, 5050, and 5173...'
                    def ports = [27017, 5050, 5173]
                    ports.each { port ->
                        def containers = sh(script: "docker ps --filter 'publish=${port}' --format '{{.ID}}'", returnStdout: true).trim()
                        containers.split("\n").each { container ->
                            if (container) {
                                echo "Attempting to remove container ${container}..."
                                def removeStatus = sh(script: "docker rm -f ${container} || true", returnStatus: true)
                                if (removeStatus != 0) {
                                    echo "Failed to remove container ${container}, retrying..."
                                    sleep(RETRY_DELAY)
                                    sh "docker rm -f ${container} || true"
                                }
                            }
                        }
                    }
                    echo 'Removing lingering backend container...'
                    sh 'docker rm -f backend || true'
                }
            }
        }

        stage('Setup Docker Network') {
            steps {
                script {
                    def networkExists = sh(script: "docker network ls --filter name=^${DOCKER_NETWORK}\$ --format '{{.Name}}'", returnStdout: true).trim()
                    if (!networkExists) {
                        sh "docker network create ${DOCKER_NETWORK}"
                        echo "Docker network '${DOCKER_NETWORK}' created."
                    } else {
                        echo "Docker network '${DOCKER_NETWORK}' already exists."
                    }
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    echo 'Running Docker Compose to build and start services...'
                    def status = sh(script: 'docker-compose -f docker-compose.yml up --build -d', returnStatus: true)
                    if (status != 0) {
                        error("Docker Compose failed with exit code ${status}")
                    }
                }
            }
        }

        stage('Verify Frontend') {
            steps {
                script {
                    echo 'Checking frontend at http://localhost:5173...'
                    def response = sh(script: 'curl -s http://localhost:5173', returnStdout: true).trim()
                    if (response.contains('<!doctype html>')) {
                        echo "✅ Frontend is running"
                    } else {
                        error "❌ Frontend is not accessible"
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                echo 'Build failed. Performing cleanup...'
                sh 'docker-compose -f docker-compose.yml down --volumes --remove-orphans || true'
                sh 'docker rm -f backend || true'
                sh "docker network rm ${DOCKER_NETWORK} || true"
            }
        }
    }
}
