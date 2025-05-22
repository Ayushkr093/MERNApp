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

        stage('Create Docker Network') {
            steps {
                script {
                    // Check if the network exists
                    def networkExists = sh(script: "docker network ls --filter name=^mern-network\$ --format {{.Name}}", returnStdout: true).trim()

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

        stage('Run Docker Compose') {
            steps {
                script {
                    try {
                        // Run docker-compose up to build and start the services (frontend, backend, mongodb)
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
    }

    post {
        failure {
            script {
                echo 'Build failed. Running cleanup actions...'

                // Clean up Docker Compose resources
                sh 'docker-compose -f docker-compose.yml down'

                // Clean up the created Docker network if needed
                sh "docker network ls --filter name=^mern-network\$ --format {{.Name}} | xargs -r docker network rm"
            }
        }
    }
}
