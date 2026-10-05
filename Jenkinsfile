pipeline {
    agent any

    environment {
        // Docker Desktop Linux Engine
        DOCKER_HOST = 'npipe:////./pipe/dockerDesktopLinuxEngine'

        IMAGE_NAME = 'jenkins-python-docker'
        CONTAINER_NAME = 'jenkins-python-app'

        HOST_PORT = '5000'
        CONTAINER_PORT = '5000'
    }

    stages {

        stage('Python Tests') {
            steps {
                echo '======================================'
                echo 'Running Python Tests'
                echo '======================================'

                bat '''
                python --version
                python -m pip install -r requirements.txt
                python -m pytest
                '''
            }
        }

        stage('Docker Check') {
            steps {
                echo '======================================'
                echo 'Checking Docker'
                echo '======================================'

                bat '''
                echo Docker Host:
                echo %DOCKER_HOST%

                echo.
                echo Docker Version:
                docker --version

                echo.
                echo Docker Info:
                docker info
                '''
            }
        }

        stage('Docker Build') {
            steps {
                echo '======================================'
                echo 'Building Docker Image'
                echo '======================================'

                bat '''
                docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .
                docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest
                '''
            }
        }

        stage('Docker Run') {
            steps {
                echo '======================================'
                echo 'Starting Docker Container'
                echo '======================================'

                bat '''
                docker rm -f %CONTAINER_NAME% 2>NUL || exit /b 0

                docker run -d ^
                    --name %CONTAINER_NAME% ^
                    -p %HOST_PORT%:%CONTAINER_PORT% ^
                    %IMAGE_NAME%:%BUILD_NUMBER%
                '''

                echo 'Docker container started successfully.'
            }
        }

        stage('Health Check') {
            steps {
                echo '======================================'
                echo 'Checking Application Health'
                echo '======================================'

                bat '''
                timeout /t 5 /nobreak >NUL

                curl -f http://localhost:%HOST_PORT%/health
                '''

                echo 'Application health check passed.'
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
            echo '======================================'
            echo 'Cleaning Up Docker Container'
            echo '======================================'

            bat '''
            docker rm -f %CONTAINER_NAME% 2>NUL || exit /b 0
            '''
        }
    }
}
