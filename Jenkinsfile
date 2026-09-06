pipeline {
    agent any

    environment {
        IMAGE_NAME = "ortofia-app"
    }

    parameters {
        choice(
            name: 'DEPLOY_ENV',
            choices: ['dev', 'prod'],
            description: 'Select Environment to Deploy'
        )
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1.0',
            description: 'Docker Image Tag/Version'
        )
        booleanParam(
            name: 'RUN_HEALTH_CHECK',
            defaultValue: true,
            description: 'Run HTTP Health Check after deploy?'
        )
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
                echo "🐳 Building Docker Image: ${IMAGE_NAME}:${params.IMAGE_TAG}"
                // Double quotes so Groovy expands variables
                sh "docker build -t ${IMAGE_NAME}:${params.IMAGE_TAG} ."
            }
        }

        stage('Conditional Deployment (if-else)') {
            steps {
                script {
                    def tag = params.IMAGE_TAG
                    def image = env.IMAGE_NAME

                    if (params.DEPLOY_ENV == 'dev') {
                        echo "🚀 Deploying to DEV Environment on Port 8083..."
                        sh """
                            docker stop veranda-dev-container || true
                            docker rm veranda-dev-container || true
                            docker run -d --name veranda-dev-container -p 8083:80 ${image}:${tag}
                        """
                    } else if (params.DEPLOY_ENV == 'prod') {
                        echo "🚀 Deploying to PROD Environment on Port 8082..."
                        sh """
                            docker stop veranda-prod-container || true
                            docker rm veranda-prod-container || true
                            docker run -d --name veranda-prod-container -p 8082:80 ${image}:${tag}
                        """
                    } else {
                        error("❌ Invalid environment selected!")
                    }
                }
            }
        }

        stage('Health Check') {
            when {
                expression { return params.RUN_HEALTH_CHECK }
            }
            steps {
                echo '🔍 Running Health Check...'
                script {
                    def targetPort = (params.DEPLOY_ENV == 'dev') ? '8083' : '8082'
                    sh "sleep 3 && curl -sI http://localhost:${targetPort} | head -n 1"
                }
            }
        }
    }

    post {
        always {
            echo '🧹 Pipeline execution completed.'
        }
        success {
            echo "🎉 SUCCESS: Deployed ${params.IMAGE_TAG} to ${params.DEPLOY_ENV}!"
        }
        failure {
            echo "❌ FAILURE: Pipeline failed for ${params.DEPLOY_ENV}!"
        }
    }
}
