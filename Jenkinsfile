pipeline {
    agent any

    environment {
        // Define environment variables
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO = 'sambathkumarj'
        DOCKER_IMAGE = 'jenkincicd'
        DOCKER_CREDENTIALS_ID = 'dockerhub'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from the repository
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Build the Docker image
                script {
                    dockerImage = docker.build("${DOCKER_REGISTRY}/${DOCKER_REPO}/${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Test') {
            steps {
                // Run your tests here
                echo 'Running tests...'
                // Example: dockerImage.inside { sh 'make test' }
            }
        }

        stage('Push') {
            steps {
                script {
                    // Push the Docker image to the registry
                    docker.withRegistry("https://${DOCKER_REGISTRY}", "${DOCKER_CREDENTIALS_ID}") {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                // Clean up the local Docker environment
                sh 'docker rmi ${DOCKER_REGISTRY}/${DOCKER_REPO}/${DOCKER_IMAGE}:${env.BUILD_NUMBER}'
            }
        }
    }

    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
