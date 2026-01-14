pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO   = "674182809289.dkr.ecr.us-east-1.amazonaws.com/weather-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t weather-app:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-creds',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                      aws ecr get-login-password --region $AWS_REGION \
                      | docker login --username AWS --password-stdin $ECR_REPO
                    '''
                }
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh '''
                  docker tag weather-app:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
                  docker push $ECR_REPO:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "Docker image pushed to ECR successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
