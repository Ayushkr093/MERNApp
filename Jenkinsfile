pipeline {
    agent any  // Run the pipeline on any available agent

    environment {
        GIT_URL = 'https://github.com/Ayushkr093/MERNApp.git' // Replace with your actual Git URL
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout the source code from Git
                git url: "${env.GIT_URL}", branch: 'main'  // You can change 'main' to your branch name
            }
        }

        stage('Verify Checkout') {
            steps {
                // List files to confirm checkout
                sh 'ls -la'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed 😢'
        }
        success {
            echo 'Pipeline succeeded ✅'
        }
    }
}
