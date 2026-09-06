pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Deploy with Docker Compose') {
            steps {
                echo '🐳 Building Docker Image & Running Container on Port 8085...'
                sh """
                    docker compose down || true
                    docker compose up -d --build
                """
            }
        }

        stage('Health Check') {
            steps {
                echo '🔍 Verifying app deployment...'
                sh """
                    sleep 3
                    docker ps | grep new-portfolio-container
                    curl -sI http://localhost:8084 | head -n 1
                """
            }
        }
    }

    post {
        success {
            echo '🎉 New Website is successfully LIVE on Port 8084!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
