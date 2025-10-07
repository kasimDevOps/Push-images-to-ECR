pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        AWS_ACCOUNT_ID = "759896056893"
        DOCKER_IMAGE = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/python-flask-app"
        GITHUB_CREDENTIALS = credentials('github-creds')  // Jenkins ID for GitHub
        AWS_CREDENTIALS = 'aws-creds'                    // Jenkins ID for AWS
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Cloning code from GitHub..."
                git branch: 'main',
                    url: 'https://github.com/kasimDevOps/Push-images-to-docker-hub-project.git',
                    credentialsId: "${GITHUB_CREDENTIALS}"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "📦 Building Docker image..."
                // Current directory has Dockerfile
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo "🔑 Logging into AWS ECR..."
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding', 
                    credentialsId: "${AWS_CREDENTIALS}"
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login --username AWS --password-stdin ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Tag and Push Image') {
            steps {
                echo "🚀 Tagging and pushing Docker image to AWS ECR..."
                sh """
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Cleanup') {
            steps {
                echo "🧹 Cleaning up local Docker images..."
                sh "docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest || true"
            }
        }
    }

    post {
        success {
            echo "✅ Image pushed successfully to AWS ECR!"
        }
        failure {
            echo "❌ Build failed. Check Jenkins logs."
        }
    }
}
