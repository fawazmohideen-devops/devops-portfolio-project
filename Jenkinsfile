pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/fawazmohideen-devops/devops-portfolio-project.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t js-app:1.0 .'
            }
        }
        stage('Verify Image') {
            steps {
                sh 'docker images'
            }
        }
    }
}