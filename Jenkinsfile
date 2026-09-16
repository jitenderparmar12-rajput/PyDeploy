pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t python-devops-app:latest .'
            }
        }

        stage('Test') {
            steps {
                sh 'docker run -d --name test-app -p 5001:5000 python-devops-app:latest'
                sh 'sleep 3'
                sh 'curl -f http://localhost:5001/health'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f deployment.yaml'
                sh 'kubectl apply -f service.yaml'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f test-app || true'
        }
    }
}