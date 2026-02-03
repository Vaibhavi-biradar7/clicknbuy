pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_NAME', defaultValue: 'myapp', description: 'Docker image name')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
        string(name: 'DOCKER_USERNAME', defaultValue: 'dockerhub_username', description: 'DockerHub username')
        password(name: 'DOCKER_PASSWORD', description: 'DockerHub password')
    }

    environment {
        FULL_IMAGE_NAME = "${params.DOCKER_USERNAME}/${params.IMAGE_NAME}:${params.IMAGE_TAG}"
    }

    stages {

        stage('Build & Push Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

                    echo "Tagging Docker image..."
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE_NAME}

                    echo "Docker login..."
                    echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin

                    echo "Pushing Docker image..."
                    docker push ${FULL_IMAGE_NAME}

                    echo "Deleting local Docker images..."
                    docker rmi ${FULL_IMAGE_NAME} || true
                    docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true

                    echo "Docker logout..."
                    docker logout
                '''
            }
        }

        stage('Docker-Compose Up') {
            steps {
                sh '''
                    echo "Starting services using Docker Compose..."
                    docker-compose up -d --build
                '''
            }
        }

        stage('Docker-Compose Down') {
            steps {
                sh '''
                    echo "Stopping services using Docker Compose..."
                    docker-compose down
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
        always {
            echo "🧹 Cleaning workspace..."
            cleans()
        }
    }
}
