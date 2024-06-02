pipeline {
    agent any

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker image tag')
    }

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '183991395055'
        ECR_REPOSITORY = 'hello-repository'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/mansuralamkhan/nodejs.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}")
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                script {
                    dockerImage.push()
                }
            }
        }

        stage('Update Kubernetes YAML') {
            steps {
                script {
                    // Fetch the current image tag from the kubernetes.yaml file
                    def oldTag = sh(script: "grep -oP '(?<=image: ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPOSITORY}:)\\S+' kubernetes.yaml", returnStdout: true).trim()

                    // Update Kubernetes YAML with the new image tag
                    sh "sed -i '' 's/${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com\\/hello-repository:${oldTag}/${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com\\/hello-repository:${params.IMAGE_TAG}/' kubernetes.yaml"

                    // Commit and push the updated Kubernetes YAML file to GitHub
                    sh 'git add kubernetes.yaml'
                    sh 'git commit -m "Update kubernetes.yaml with new image tag ${params.IMAGE_TAG}"'
                    sh 'git push origin master'
                }
            }
        }


        stage('Deploy to Kubernetes') {
            steps {
                // Deploy the application to Kubernetes
                sh 'kubectl apply -f kubernetes.yaml'
            }
        }
    }

    post {
        always {
            script {
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}"
            }
        }
    }
}
