pipeline {
    agent any
    environment {
        ECR_REGISTRY = '443370696188.dkr.ecr.ca-central-1.amazonaws.com'
        ECR_REPO = 'js-app'
        AWS_REGION = 'ca-central-1'
        IMAGE_TAG = '1.0'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/fawazmohideen-devops/devops-portfolio-project.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }
        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REGISTRY'
            }
        }
        stage('Tag and Push Image') {
            steps {
                sh 'docker tag $ECR_REPO:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG'
                sh 'docker push $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG'
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker rm -f js-app-container || true'
                sh 'docker run -d --name js-app-container -p 3000:3000 $ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG'
            }
        }
    }
}