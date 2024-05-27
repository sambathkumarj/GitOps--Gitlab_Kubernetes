pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {    
                    echo 'Building test...' 
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
            }
        }
