pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        DOCKER_NETWORK = 'mern-network'
        COMPOSE_PROJECT_NAME = 'mernapp'
        FRONTEND_PORT = '5173'
        BACKEND_PORT = '5050'
        MONGO_PORT = '27017'
        // Health check configuration
        HEALTH_CHECK_TIMEOUT = '300' // 5 minutes
        HEALTH_CHECK_INTERVAL = '10' // 10 seconds
    }

    options {
        // Keep only last 10 builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        // Timeout the entire pipeline after 30 minutes
        timeout(time: 30, unit: 'MINUTES')
        // Add timestamps to console output
        timestamps()
        // Prevent concurrent builds
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    try {
                        echo "Checking out code from ${env.GIT_URL}"
                        checkout scm
                    } catch (Exception e) {
                        error "Failed to checkout code: ${e.getMessage()}"
                    }
                }
            }
        }

        stage('Validate Environment') {
            steps {
                script {
                    echo 'Validating Docker and Docker Compose installation...'
                    sh '''
                        docker --version
                        docker-compose --version
                        
                        # Check if required files exist
                        if [ ! -f "docker-compose.yml" ]; then
                            echo "ERROR: docker-compose.yml not found in repository"
                            exit 1
                        fi
                        
                        echo "Environment validation completed successfully"
                    '''
                }
            }
        }

        stage('Pre-deployment Cleanup') {
            steps {
                script {
                    echo 'Performing comprehensive cleanup before deployment...'
                    sh '''
                        # Stop and remove containers using our project name
                        docker-compose -p ${COMPOSE_PROJECT_NAME} down --volumes --remove-orphans || true
                        
                        # Clean up any containers using our specific ports
                        for port in ${MONGO_PORT} ${BACKEND_PORT} ${FRONTEND_PORT}; do
                            CONTAINER_ID=$(docker ps -q --filter "publish=${port}")
                            if [ ! -z "$CONTAINER_ID" ]; then
                                echo "Stopping container using port ${port}: $CONTAINER_ID"
                                docker stop $CONTAINER_ID || true
                                docker rm $CONTAINER_ID || true
                            fi
                        done
                        
                        # Remove any dangling containers with our project prefix
                        docker ps -a --filter "name=${COMPOSE_PROJECT_NAME}" -q | xargs -r docker rm -f || true
                        
                        # Clean up unused images and volumes (optional - uncomment if needed)
                        # docker system prune -f --volumes
                        
                        echo "Cleanup completed successfully"
                    '''
                }
            }
        }

        stage('Setup Docker Network') {
            steps {
                script {
                    echo "Setting up Docker network: ${DOCKER_NETWORK}"
                    sh '''
                        # Check if network exists
                        if ! docker network ls --format "{{.Name}}" | grep -q "^${DOCKER_NETWORK}$"; then
                            echo "Creating Docker network: ${DOCKER_NETWORK}"
                            docker network create ${DOCKER_NETWORK}
                        else
                            echo "Docker network '${DOCKER_NETWORK}' already exists"
                        fi
                        
                        # Verify network was created/exists
                        docker network inspect ${DOCKER_NETWORK} > /dev/null
                        echo "Network setup completed successfully"
                    '''
                }
            }
        }

        stage('Build and Deploy Services') {
            steps {
                script {
                    echo 'Building and starting services with Docker Compose...'
                    sh '''
                        # Use project name to avoid conflicts
                        docker-compose -p ${COMPOSE_PROJECT_NAME} -f docker-compose.yml up --build -d
                        
                        # Wait a moment for containers to initialize
                        sleep 10
                        
                        # Show running containers for debugging
                        echo "Currently running containers:"
                        docker-compose -p ${COMPOSE_PROJECT_NAME} ps
                    '''
                }
            }
        }

        stage('Health Checks') {
            parallel {
                stage('Backend Health Check') {
                    steps {
                        script {
                            echo 'Checking backend service health...'
                            timeout(time: 5, unit: 'MINUTES') {
                                waitUntil {
                                    script {
                                        try {
                                            def response = sh(
                                                script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:${BACKEND_PORT}/health || echo '000'",
                                                returnStdout: true
                                            ).trim()
                                            
                                            if (response == '200') {
                                                echo "Backend health check passed (HTTP ${response})"
                                                return true
                                            } else {
                                                echo "Backend health check failed (HTTP ${response}), retrying..."
                                                sleep(10)
                                                return false
                                            }
                                        } catch (Exception e) {
                                            echo "Backend health check error: ${e.getMessage()}, retrying..."
                                            sleep(10)
                                            return false
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
                
                stage('Frontend Health Check') {
                    steps {
                        script {
                            echo 'Checking frontend service health...'
                            timeout(time: 5, unit: 'MINUTES') {
                                waitUntil {
                                    script {
                                        try {
                                            def response = sh(
                                                script: "curl -s http://localhost:${FRONTEND_PORT}",
                                                returnStdout: true
                                            )
                                            
                                            if (response.contains("<!doctype html>") || response.contains("<!DOCTYPE html>")) {
                                                echo "Frontend is accessible and serving HTML content"
                                                return true
                                            } else {
                                                echo "Frontend health check failed, retrying..."
                                                sleep(10)
                                                return false
                                            }
                                        } catch (Exception e) {
                                            echo "Frontend health check error: ${e.getMessage()}, retrying..."
                                            sleep(10)
                                            return false
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
                
                stage('Database Health Check') {
                    steps {
                        script {
                            echo 'Checking database connectivity...'
                            timeout(time: 3, unit: 'MINUTES') {
                                waitUntil {
                                    script {
                                        try {
                                            // Check if MongoDB port is responding
                                            def result = sh(
                                                script: "nc -z localhost ${MONGO_PORT}",
                                                returnStatus: true
                                            )
                                            
                                            if (result == 0) {
                                                echo "Database port ${MONGO_PORT} is accessible"
                                                return true
                                            } else {
                                                echo "Database health check failed, retrying..."
                                                sleep(10)
                                                return false
                                            }
                                        } catch (Exception e) {
                                            echo "Database health check error: ${e.getMessage()}, retrying..."
                                            sleep(10)
                                            return false
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }

        stage('Post-Deployment Verification') {
            steps {
                script {
                    echo 'Running post-deployment verification...'
                    sh '''
                        echo "=== Container Status ==="
                        docker-compose -p ${COMPOSE_PROJECT_NAME} ps
                        
                        echo "=== Container Logs (last 20 lines) ==="
                        docker-compose -p ${COMPOSE_PROJECT_NAME} logs --tail=20
                        
                        echo "=== Port Usage Verification ==="
                        netstat -tulpn | grep -E "(${FRONTEND_PORT}|${BACKEND_PORT}|${MONGO_PORT})" || echo "No ports found (this might be normal)"
                        
                        echo "=== Service URLs ==="
                        echo "Frontend: http://localhost:${FRONTEND_PORT}"
                        echo "Backend: http://localhost:${BACKEND_PORT}"
                        echo "MongoDB: localhost:${MONGO_PORT}"
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                echo 'Pipeline execution completed. Collecting logs and artifacts...'
                
                // Collect container logs
                try {
                    sh '''
                        mkdir -p logs
                        docker-compose -p ${COMPOSE_PROJECT_NAME} logs > logs/docker-compose.log 2>&1 || true
                    '''
                    archiveArtifacts artifacts: 'logs/*.log', fingerprint: true, allowEmptyArchive: true
                } catch (Exception e) {
                    echo "Failed to collect logs: ${e.getMessage()}"
                }
            }
        }
        
        success {
            script {
                echo '✅ Deployment completed successfully!'
                echo "Services are available at:"
                echo "- Frontend: http://localhost:${FRONTEND_PORT}"
                echo "- Backend: http://localhost:${BACKEND_PORT}"
                
                // Optional: Send success notification
                // slackSend channel: '#deployments', 
                //           message: "✅ MERN App deployed successfully on ${env.NODE_NAME}"
            }
        }
        
        failure {
            script {
                echo '❌ Deployment failed. Performing cleanup...'
                
                // Comprehensive cleanup on failure
                try {
                    sh '''
                        echo "Stopping and removing all services..."
                        docker-compose -p ${COMPOSE_PROJECT_NAME} down --volumes --remove-orphans || true
                        
                        echo "Force removing any remaining containers..."
                        docker ps -a --filter "name=${COMPOSE_PROJECT_NAME}" -q | xargs -r docker rm -f || true
                        
                        echo "Cleaning up network if it exists and is not in use..."
                        if docker network ls --format "{{.Name}}" | grep -q "^${DOCKER_NETWORK}$"; then
                            docker network rm ${DOCKER_NETWORK} || echo "Network ${DOCKER_NETWORK} is still in use or removal failed"
                        fi
                        
                        echo "Cleanup completed"
                    '''
                } catch (Exception e) {
                    echo "Error during cleanup: ${e.getMessage()}"
                }
                
                // Optional: Send failure notification
                // slackSend channel: '#deployments', 
                //           color: 'danger',
                //           message: "❌ MERN App deployment failed on ${env.NODE_NAME}. Check console output for details."
            }
        }
        
        unstable {
            script {
                echo '⚠️ Build completed with warnings'
            }
        }
    }
}
