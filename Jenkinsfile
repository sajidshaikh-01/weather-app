pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO   = "674182809289.dkr.ecr.ap-south-1.amazonaws.com/weather-app"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        GITOPS_REPO = "https://github.com/sajidshaikh-01/weather-gitops.git"
    }

    stages {

        stage('Checkout App Code') {
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
              aws ecr get-login-password --region us-east-1 \
              | docker login --username AWS --password-stdin 674182809289.dkr.ecr.us-east-1.amazonaws.com
            '''
        }
    }
}

        stage('Push Image to ECR') {
            steps {
                sh '''
                  docker tag weather-app:${IMAGE_TAG} $ECR_REPO:${IMAGE_TAG}
                  docker push $ECR_REPO:${IMAGE_TAG}
                '''
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'gitops-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                      rm -rf weather-gitops
                      git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/<your-username>/weather-gitops.git
                      cd weather-gitops/base

                      sed -i "s|image: .*weather-app:.*|image: $ECR_REPO:${IMAGE_TAG}|g" deployment.yaml

                      git config user.email "jenkins@ci.local"
                      git config user.name "jenkins-ci"

                      git add deployment.yaml
                      git commit -m "Update weather-app image to ${IMAGE_TAG}"
                      git push origin main
                    '''
                }
            }
        }
    }
}

