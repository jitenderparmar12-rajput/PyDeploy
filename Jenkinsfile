pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

        DOCKER_IMAGE = 'jitender12/pydeploy'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Check Docker') {
            steps {
                echo 'Checking Docker installation...'

                bat '"%DOCKER%" --version'
                bat '"%DOCKER%" ps'
            }
        }

        stage('Check Kubernetes') {
            steps {
                echo 'Checking Kubernetes installation...'

                bat '"%KUBECTL%" version --client'
                bat '"%KUBECTL%" get nodes'
            }
        }

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'

                bat '"%DOCKER%" build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
                bat '"%DOCKER%" tag %DOCKER_IMAGE%:%IMAGE_TAG% %DOCKER_IMAGE%:latest'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing Flask application...'

                bat '"%DOCKER%" run -d --name pydeploy-test -p 5001:5000 %DOCKER_IMAGE%:%IMAGE_TAG%'

                powershell '''
                    Start-Sleep -Seconds 5
                '''

                retry(3) {
                    bat 'curl.exe --fail http://localhost:5001/health'

                    powershell '''
                        Start-Sleep -Seconds 2
                    '''
                }

                bat '"%DOCKER%" stop pydeploy-test'
                bat '"%DOCKER%" rm pydeploy-test'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes...'

                bat '"%KUBECTL%" apply -f deployment.yaml'
                bat '"%KUBECTL%" apply -f service.yaml'
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                echo 'Verifying Kubernetes deployment...'

                bat '"%KUBECTL%" get deployments'
                bat '"%KUBECTL%" get pods'
                bat '"%KUBECTL%" get services'
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
                        echo %DOCKER_PASSWORD% | "%DOCKER%" login -u %DOCKER_USERNAME% --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'

                bat '"%DOCKER%" push %DOCKER_IMAGE%:%IMAGE_TAG%'
                bat '"%DOCKER%" push %DOCKER_IMAGE%:latest'
            }
        }
    }

    post {
        always {
            bat '"%DOCKER%" rm -f pydeploy-test 2>NUL || exit 0'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}