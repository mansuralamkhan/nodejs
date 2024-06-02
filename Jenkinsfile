pipeline {
    agent 

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from the Git repository
                git 'https://github.com/mansuralamkhan/nodejs.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                // Install Node.js dependencies
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // Run tests if available
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image
                sh 'docker build -t nodejs-app .'
            }
        }

        stage('Push to Docker Registry') {
            steps {
                // Push Docker image to Docker registry (e.g., Docker Hub, AWS ECR)
                sh 'docker tag nodejs-app mansuralam44/nodejs-app:latest'
                sh 'docker push mansuralam44/nodejs-app:latest'
            }
        }

        stage('Deploy') {
            steps {
                // Deploy the application (e.g., using Kubernetes)
                // This stage assumes you have Kubernetes configured in Jenkins
                // Replace 'kubectl' with the appropriate command for your deployment setup
                sh 'kubectl apply -f kubernetes.yaml'
            }
        }
    }

    post {
        always {
            // Clean up after the pipeline execution
            cleanWs()
        }
    }
}
