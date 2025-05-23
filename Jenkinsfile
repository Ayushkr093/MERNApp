pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        DOCKER_NETWORK = 'mern-network'
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
                    sh '''
                        for port in 27017 5050 5173; do
                            docker ps --filter "publish=${port}" --format "{{.ID}}" | xargs -r docker rm -f
                        done
                        docker rm -f backend || true
                    '''
                }
            }
        }

        stage('Setup Docker Network') {
            steps {
                script {
                    def networkExists = sh(script: "docker network ls --filter name=^${DOCKER_NETWORK}\$ --format '{{.Name}}'", returnStdout: true).trim()
                    if (!networkExists) {
                        sh "docker network create ${DOCKER_NETWORK}"
                        echo "Network '${DOCKER_NETWORK}' created."
                    } else {
                        echo "Network '${DOCKER_NETWORK}' already exists."
                    }
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                script {
                    echo 'Building and starting services with Docker Compose...'
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
