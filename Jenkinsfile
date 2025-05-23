pipeline {
    agent any
    
    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git'
        GIT_BRANCH = 'main'
        FRONTEND_IMAGE = 'ayushkr08/mernapp-frontend'
        BACKEND_IMAGE = 'ayushkr08/mernapp-backend'
        MONGO_IMAGE = 'mongo:latest'
        BUILD_TAG = "v1-${env.BUILD_NUMBER}"
        DOCKER_BUILDKIT = '1'  // Enable BuildKit for faster builds
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Checking out code from ${GIT_BRANCH}..."
                git branch: "${GIT_BRANCH}", url: "${GIT_URL}"
            }
        }
        
        stage('Docker System Info') {
            steps {
                echo "📊 Checking Docker system status..."
                sh """
                    docker --version
                    docker system df
                    echo "Testing Docker Hub connectivity..."
                    curl -I https://index.docker.io/v1/ || echo "Connection issue detected"
                """
            }
        }
        
        stage('Build Docker Images') {
            parallel {
                stage('Build Frontend') {
                    steps {
                        echo "🏗️ Building Frontend Docker Image..."
                        sh """
                            cd mern/frontend
                            # Build with optimizations
                            DOCKER_BUILDKIT=1 docker build \\
                                --build-arg BUILDKIT_INLINE_CACHE=1 \\
                                --cache-from ${FRONTEND_IMAGE}:latest \\
                                -t ${FRONTEND_IMAGE}:${BUILD_TAG} .
                            docker tag ${FRONTEND_IMAGE}:${BUILD_TAG} ${FRONTEND_IMAGE}:latest
                        """
                    }
                }
                stage('Build Backend') {
                    steps {
                        echo "🏗️ Building Backend Docker Image..."
                        sh """
                            cd mern/backend
                            # Build with optimizations
                            DOCKER_BUILDKIT=1 docker build \\
                                --build-arg BUILDKIT_INLINE_CACHE=1 \\
                                --cache-from ${BACKEND_IMAGE}:latest \\
                                -t ${BACKEND_IMAGE}:${BUILD_TAG} .
                            docker tag ${BACKEND_IMAGE}:${BUILD_TAG} ${BACKEND_IMAGE}:latest
                        """
                    }
                }
            }
        }
        
        stage('Docker Login') {
            steps {
                echo "🔐 Logging into Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        echo "✅ Docker login successful"
                    '''
                }
            }
        }
        
        stage('Push Docker Images') {
            parallel {
                stage('Push Frontend Images') {
                    steps {
                        echo "🚀 Pushing Frontend images to Docker Hub..."
                        script {
                            // Check if versioned image already exists
                            def frontendExists = sh(
                                script: "docker manifest inspect ${FRONTEND_IMAGE}:${BUILD_TAG} > /dev/null 2>&1",
                                returnStatus: true
                            )
                            
                            if (frontendExists != 0) {
                                sh """
                                    echo "Pushing new frontend image..."
                                    docker push ${FRONTEND_IMAGE}:${BUILD_TAG}
                                    docker push ${FRONTEND_IMAGE}:latest
                                """
                            } else {
                                echo "Frontend image ${BUILD_TAG} already exists, skipping push"
                            }
                        }
                    }
                }
                stage('Push Backend Images') {
                    steps {
                        echo "🚀 Pushing Backend images to Docker Hub..."
                        script {
                            // Check if versioned image already exists
                            def backendExists = sh(
                                script: "docker manifest inspect ${BACKEND_IMAGE}:${BUILD_TAG} > /dev/null 2>&1",
                                returnStatus: true
                            )
                            
                            if (backendExists != 0) {
                                sh """
                                    echo "Pushing new backend image..."
                                    docker push ${BACKEND_IMAGE}:${BUILD_TAG}
                                    docker push ${BACKEND_IMAGE}:latest
                                """
                            } else {
                                echo "Backend image ${BUILD_TAG} already exists, skipping push"
                            }
                        }
                    }
                }
            }
        }
        
        stage('Cleanup Old Containers') {
            steps {
                echo "🧹 Cleaning up existing containers and networks..."
                sh """
                    # Stop and remove existing containers if they exist
                    docker stop frontend backend mongodb 2>/dev/null || true
                    docker rm frontend backend mongodb 2>/dev/null || true
                    
                    # Remove existing network if it exists
                    docker network rm demo 2>/dev/null || true
                    
                    # Clean up unused images to free space
                    docker image prune -f
                """
            }
        }
        
        stage('Setup and Run Containers') {
            steps {
                echo "⚙️ Setting up network and running containers..."
                sh """
                    # Create the custom network
                    docker network create demo
                    
                    # Run MongoDB container with restart policy
                    docker run \\
                        --network=demo \\
                        --name mongodb \\
                        --restart=unless-stopped \\
                        -d \\
                        -p 27017:27017 \\
                        -v ~/opt/data:/data/db \\
                        ${MONGO_IMAGE}
                    
                    # Wait for MongoDB to be ready
                    echo "Waiting for MongoDB to start..."
                    sleep 10
                    
                    # Run Backend container
                    docker run \\
                        --name=backend \\
                        --network=demo \\
                        --restart=unless-stopped \\
                        -d \\
                        -p 5050:5050 \\
                        ${BACKEND_IMAGE}:${BUILD_TAG}
                    
                    # Wait for backend to be ready
                    echo "Waiting for Backend to start..."
                    sleep 5
                    
                    # Run Frontend container
                    docker run \\
                        --name=frontend \\
                        --network=demo \\
                        --restart=unless-stopped \\
                        -d \\
                        -p 5173:5173 \\
                        ${FRONTEND_IMAGE}:${BUILD_TAG}
                    
                    # Wait for services to be ready
                    echo "Waiting for services to be fully ready..."
                    sleep 10
                """
            }
        }
        
        stage('Health Check') {
            steps {
                echo "🏥 Performing health checks..."
                sh """
                    # Check container status
                    echo "=== Container Status ==="
                    docker ps --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                    
                    # Basic connectivity tests
                    echo "\\n=== Health Checks ==="
                    
                    # Check if containers are running
                    if docker ps | grep -q mongodb; then
                        echo "✅ MongoDB container is running"
                    else
                        echo "❌ MongoDB container failed to start"
                        exit 1
                    fi
                    
                    if docker ps | grep -q backend; then
                        echo "✅ Backend container is running"
                    else
                        echo "❌ Backend container failed to start"
                        exit 1
                    fi
                    
                    if docker ps | grep -q frontend; then
                        echo "✅ Frontend container is running"
                    else
                        echo "❌ Frontend container failed to start"
                        exit 1
                    fi
                    
                    # Test port accessibility
                    echo "\\n=== Port Accessibility ==="
                    netstat -tlnp | grep :5173 && echo "✅ Frontend port 5173 is accessible" || echo "⚠️ Frontend port 5173 not accessible"
                    netstat -tlnp | grep :5050 && echo "✅ Backend port 5050 is accessible" || echo "⚠️ Backend port 5050 not accessible"
                    netstat -tlnp | grep :27017 && echo "✅ MongoDB port 27017 is accessible" || echo "⚠️ MongoDB port 27017 not accessible"
                    
                    # Optional: Test HTTP endpoints (uncomment if your apps support it)
                    # curl -f http://localhost:5173 > /dev/null 2>&1 && echo "✅ Frontend HTTP check passed" || echo "⚠️ Frontend HTTP check failed"
                    # curl -f http://localhost:5050/health > /dev/null 2>&1 && echo "✅ Backend HTTP check passed" || echo "⚠️ Backend HTTP check failed"
                """
            }
        }
    }
    
    post {
        always {
            echo "🔍 Pipeline execution completed. Gathering final information..."
            sh """
                echo "=== Final Container Status ==="
                docker ps -a --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                
                echo "\\n=== Docker System Usage ==="
                docker system df
                
                echo "\\n=== Network Information ==="
                docker network ls | grep demo || echo "Demo network not found"
            """
        }
        success {
            echo '''
            ✅ 🎉 PIPELINE SUCCESS! 🎉 ✅
            
            Your MERN application is now running:
            - Frontend: http://localhost:5173
            - Backend: http://localhost:5050  
            - MongoDB: localhost:27017
            
            Images pushed to Docker Hub:
            - ${FRONTEND_IMAGE}:${BUILD_TAG} & latest
            - ${BACKEND_IMAGE}:${BUILD_TAG} & latest
            '''
        }
        failure {
            echo '''
            ❌ PIPELINE FAILED ❌
            
            Check the logs above for specific error details.
            Common issues:
            - Docker Hub authentication
            - Port conflicts
            - Network connectivity
            - Insufficient disk space
            '''
            
            // Cleanup on failure
            sh """
                echo "Cleaning up after failure..."
                docker stop frontend backend mongodb 2>/dev/null || true
                docker rm frontend backend mongodb 2>/dev/null || true
                docker network rm demo 2>/dev/null || true
            """
        }
        cleanup {
            echo "🧹 Performing final cleanup..."
            sh """
                # Logout from Docker Hub
                docker logout || true
                
                # Clean up build cache (optional - comment out if you want to keep cache)
                # docker builder prune -f
            """
        }
    }
}
