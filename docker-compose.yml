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

        stage('Initial Cleanup (Avoid Port Conflicts)') {
            steps {
                script {
                    echo 'Cleaning up containers using ports 27017, 5050, and 5173...'
                    sh '''
                    for port in 27017 5050 5173; do
                        CONTAINER=$(docker ps --filter "publish=${port}" --format "{{.ID}}")
                        if [ ! -z "$CONTAINER" ]; then
                            docker stop $CONTAINER && docker rm $CONTAINER
                        fi
                    done

                    echo "Removing any existing 'backend' container to release network endpoints..."
                    docker rm -f backend || true
                    '''
                }
            }
        }

        stage('Create Docker Network') {
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
                    try {
                        echo 'Running Docker Compose to build and start services...'
                        sh 'docker-compose -f docker-compose.yml up --build -d'
                    } catch (Exception e) {
                        echo "Docker Compose failed: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE'
                        return
                    }
                }
            }
        }

        stage('Verify Frontend is Running') {
            steps {
                script {
                    try {
                        def response = sh(script: 'curl -s http://localhost:5173', returnStdout: true)
                        if (response.contains("<!doctype html>")) {
                            echo "Frontend is running at http://localhost:5173"
                        } else {
                            error "Unexpected response from frontend."
                        }
                    } catch (Exception e) {
                        echo "Frontend is not accessible: ${e.getMessage()}"
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
    }

    post {
        failure {
            script {
                echo 'Build failed. Running cleanup actions...'

                try {
                    sh 'docker-compose -f docker-compose.yml down --volumes --remove-orphans'
                } catch (Exception e) {
                    echo "Error during docker-compose down: ${e.getMessage()}"
                }

                try {
                    echo 'Force removing lingering backend container...'
                    sh 'docker rm -f backend || true'
                } catch (Exception e) {
                    echo "Error removing backend container: ${e.getMessage()}"
                }

                try {
                    echo 'Removing custom Docker network if possible...'
                    sh "docker network rm ${DOCKER_NETWORK} || true"
                } catch (Exception e) {
                    echo "Error removing network: ${e.getMessage()}"
                }
            }
        }
    }
}
