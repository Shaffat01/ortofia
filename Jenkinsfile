pipeline {
    agent any

    environment {
        // ⚠️ তোমার আসল Docker Hub Username বসাও
        DOCKER_USER = 'shaffat01'
        IMAGE_NAME = 'veranda-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}" // প্রতি বিল্ডে আলাদা ট্যাগ হবে (v1, v2, v3...)
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Pulling code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Image: ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                sh "docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                sh "docker tag ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKER_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '🔐 Logging in to Docker Hub & Pushing Image...'
                // Jenkins Credentials Manager থেকে নিরাপদভাবে লগইন করা
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials', 
                    passwordVariable: 'DOCKER_PASS', 
                    usernameVariable: 'DOCKER_USER_ENV'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER_ENV --password-stdin'
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                echo '🚀 Deploying container from Docker Hub on Port 8085...'
                sh """
                    docker stop veranda-hub-container || true
                    docker rm veranda-hub-container || true
                    docker run -d --name veranda-hub-container -p 8085:80 ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            echo '🧹 Cleaning up local dangling images...'
            sh 'docker image prune -f || true'
        }
        success {
            echo "🎉 SUCCESS: Image pushed to Docker Hub and App Live on Port 8085!"
        }
        failure {
            echo "❌ FAILURE: Docker Hub Push or Deployment failed!"
        }
    }
}
