pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'jitender12/pydeploy'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
                bat 'docker tag %DOCKER_IMAGE%:%IMAGE_TAG% %DOCKER_IMAGE%:latest'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing Flask application...'

                bat 'docker run -d --name pydeploy-test -p 5001:5000 %DOCKER_IMAGE%:%IMAGE_TAG%'
                bat 'timeout /t 5 /nobreak'
                bat 'curl http://localhost:5001/health'
                bat 'docker stop pydeploy-test'
                bat 'docker rm pydeploy-test'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes...'

                bat 'kubectl apply -f deployment.yaml'
                bat 'kubectl apply -f service.yaml'
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                echo 'Verifying Kubernetes deployment...'

                bat 'kubectl get deployments'
                bat 'kubectl get pods'
                bat 'kubectl get services'
            }
        }

        stage('DockerHub Login') {
            steps {
                echo 'Logging in to DockerHub...'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'

                bat 'docker push %DOCKER_IMAGE%:%IMAGE_TAG%'
                bat 'docker push %DOCKER_IMAGE%:latest'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }

        always {
            bat 'docker rm -f pydeploy-test 2>NUL || exit 0'
        }
    }
}