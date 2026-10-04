pipeline {

    agent any

    environment {
        IMAGE_NAME = "jenkins-python-docker"
        CONTAINER_NAME = "jenkins-python-app"
        HOST_PORT = "5000"
        CONTAINER_PORT = "5000"
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Python Tests') {
            steps {
                echo 'Running Python tests...'

                bat '''
                python -m pip install -r requirements.txt
                python -m pytest
                '''
            }
        }

        stage('Docker Check') {
            steps {
                echo 'Checking Docker...'

                bat '''
                docker --version
                docker info
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'

                bat '''
                docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                '''
            }
        }

        stage('Docker Run') {
            steps {
                echo 'Starting Docker container...'

                bat '''
                docker rm -f %CONTAINER_NAME% 2>NUL || exit /b 0

                docker run -d ^
                    --name %CONTAINER_NAME% ^
                    -p %HOST_PORT%:%CONTAINER_PORT% ^
                    %IMAGE_NAME%:%BUILD_NUMBER%
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Checking application health...'

                bat '''
                timeout /t 5 /nobreak >NUL
                curl -f http://localhost:%HOST_PORT%/health
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'JENKINS + DOCKER PIPELINE SUCCESSFUL'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo 'JENKINS + DOCKER PIPELINE FAILED'
            echo '======================================'
        }

        always {
            echo 'Cleaning up Docker container...'

            bat '''
            docker rm -f %CONTAINER_NAME% 2>NUL || exit /b 0
            '''
        }
    }
}