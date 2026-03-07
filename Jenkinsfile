pipeline {
    agent any

    environment {
        GITHUB_CREDS = credentials('github-creds')
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "vimalmahalingam/ci-cd-pipeline"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/vimalmahalingam/CI-CD-Pipeline.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Build & Test') {
            steps {
                bat 'python -m unittest test_app.py'
            }
        }

        stage('Set Docker Context') {
            steps {
                bat 'docker context use desktop-linux'
            }
        }

        stage('Docker Build') {
            steps {
                bat "docker build -t %DOCKER_IMAGE%:${env.BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                bat "echo %DOCKERHUB_CREDS_PSW% | docker login -u %DOCKERHUB_CREDS_USR% --password-stdin"
                bat "docker push %DOCKER_IMAGE%:${env.BUILD_NUMBER}"
            }
        }

        stage('Deploy') {
            steps {
                bat "docker run -d -p 8080:8080 %DOCKER_IMAGE%:${env.BUILD_NUMBER}"
            }
        }
    }
}
