pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'

        KUBECTL = 'C:\\Users\\JITENDER PARMAR\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'

        KUBE_CONFIG = 'C:\\Users\\JITENDER PARMAR\\.kube\\config'

        DOCKER_IMAGE = 'jitender12/pydeploy'

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
            }
        }

        stage('Check Docker') {
            steps {
                echo 'Checking Docker installation...'

                bat '"%DOCKER%" --version'

                bat '"%DOCKER%" ps'
            }
        }

        stage('Check Kubernetes Access') {
            steps {
                echo 'Checking Kubernetes configuration...'

                bat 'echo Kubeconfig path: %KUBE_CONFIG%'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" config current-context'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" cluster-info'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" get nodes'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

                bat '"%DOCKER%" build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'

                bat '"%DOCKER%" tag %DOCKER_IMAGE%:%IMAGE_TAG% %DOCKER_IMAGE%:latest'
            }
        }

        stage('Test Application') {
            steps {
                echo 'Starting test container...'

                bat '"%DOCKER%" rm -f pydeploy-test 2>NUL || exit 0'

                bat '"%DOCKER%" run -d --name pydeploy-test -p 5001:5000 %DOCKER_IMAGE%:%IMAGE_TAG%'

                powershell '''
                    Start-Sleep -Seconds 8
                '''

                echo 'Testing application health endpoint...'

                retry(3) {
                    bat 'curl.exe --fail http://localhost:5001/health'

                    powershell '''
                        Start-Sleep -Seconds 2
                    '''
                }

                echo 'Application test successful.'
            }

            post {
                always {
                    bat '"%DOCKER%" stop pydeploy-test 2>NUL || exit 0'

                    bat '"%DOCKER%" rm pydeploy-test 2>NUL || exit 0'
                }
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
                echo "Pushing Docker image: ${DOCKER_IMAGE}:${IMAGE_TAG}"

                bat '"%DOCKER%" push %DOCKER_IMAGE%:%IMAGE_TAG%'

                bat '"%DOCKER%" push %DOCKER_IMAGE%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Applying Kubernetes deployment...'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" apply -f deployment.yaml'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" apply -f service.yaml'
            }
        }

        stage('Update Kubernetes Image') {
            steps {
                echo "Updating Kubernetes deployment image to ${DOCKER_IMAGE}:${IMAGE_TAG}"

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" set image deployment/pydeploy pydeploy=%DOCKER_IMAGE%:%IMAGE_TAG%'
            }
        }

        stage('Wait for Kubernetes Rollout') {
            steps {
                echo 'Waiting for Kubernetes rollout...'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" rollout status deployment/pydeploy --timeout=180s'
            }
        }

        stage('Verify Kubernetes Deployment') {
            steps {
                echo 'Checking Kubernetes deployment...'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" get deployments'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" get pods -o wide'

                bat '"%KUBECTL%" --kubeconfig "%KUBE_CONFIG%" get services'
            }
        }
    }

    post {
        always {
            echo 'Performing cleanup...'

            bat '"%DOCKER%" rm -f pydeploy-test 2>NUL || exit 0'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Please check the Jenkins console output.'
        }
    }
}