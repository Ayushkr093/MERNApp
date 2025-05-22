pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' 
        FRONTEND_IMAGE = 'mern-frontend'
        BACKEND_IMAGE = 'mern-backend'
        DOCKER_NETWORK = 'mern-network'
        FRONTEND_PORT = '5173' 
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
            // Check if the network exists
            def networkExists = sh(script: "docker network ls --filter name=^mern-network$ --format {{.Name}}", returnStdout: true).trim()

            // Create network if it does not exist
            if (!networkExists) {
                sh 'docker network create mern-network'
                echo 'Docker network "mern-network" created.'
            } else {
                echo 'Docker network "mern-network" already exists.'
            }
        }
    }
}

stage('Build Frontend') {
    steps {
        dir('mern/frontend') {
            script {
                try {
                    sh 'docker build -t mern-frontend .'
                } catch (Exception e) {
                    echo "Build failed: ${e.getMessage()}"
                    currentBuild.result = 'FAILURE'
                    return
                }
            }
        }
    }
}

stage('Run Frontend Container') {
    steps {
        script {
            try {
                // Clean up existing frontend container if needed
                sh "docker ps -a -q -f name=frontend | xargs -r docker rm -f"

                // Run the frontend container
                sh 'docker run --name=frontend --network=mern-network -d -p 5173:5173 mern-frontend'
            } catch (Exception e) {
                echo "Failed to start frontend container: ${e.getMessage()}"
                currentBuild.result = 'FAILURE'
                return
            }
        }
    }
}

stage('Run MongoDB Container') {
    steps {
        script {
            try {
                // Clean up any existing MongoDB container
                sh "docker ps -a -q -f name=mongodb | xargs -r docker rm -f"
                
                // Run the MongoDB container
                sh 'docker run --network=mern-network --name=mongodb -d -p 27017:27017 mongo:latest'
            } catch (Exception e) {
                echo "Failed to start MongoDB container: ${e.getMessage()}"
                currentBuild.result = 'FAILURE'
                return
            }
        }
    }
}

stage('Build Backend') {
    steps {
        dir('mern/backend') {
            script {
                try {
                    sh 'docker build -t mern-backend .'
                } catch (Exception e) {
                    echo "Backend build failed: ${e.getMessage()}"
                    currentBuild.result = 'FAILURE'
                    return
                }
            }
        }
    }
}

stage('Run Backend Container') {
    steps {
        script {
            try {
                // Clean up existing backend container if needed
                sh "docker ps -a -q -f name=backend | xargs -r docker rm -f"

                // Run the backend container
                sh 'docker run --name=backend --network=mern-network -d -p 5050:5050 mern-backend'
            } catch (Exception e) {
                echo "Failed to start backend container: ${e.getMessage()}"
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
                // Check if frontend is accessible
                sh 'curl -s http://localhost:5173'
                echo "Frontend is running at http://localhost:5173"
            } catch (Exception e) {
                echo "Frontend is not accessible: ${e.getMessage()}"
                currentBuild.result = 'FAILURE'
            }
        }
    }
}

stage('Post Actions') {
    steps {
        script {
            echo 'Cleaning up Docker resources...'
            // Remove containers if they exist
            sh "docker ps -a -q -f name=frontend | xargs -r docker rm -f"
            sh "docker ps -a -q -f name=backend | xargs -r docker rm -f"
            sh "docker ps -a -q -f name=mongodb | xargs -r docker rm -f"
            
            // Remove network if needed
            sh "docker network ls --filter name=^mern-network$ --format {{.Name}} | xargs -r docker network rm"
        }
    }
}
