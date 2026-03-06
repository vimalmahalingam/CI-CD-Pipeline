pipeline {
    agent any

    environment {
        // GitHub credentials (username/password or token)
        GITHUB_CREDS = credentials('github-creds')

        // DockerHub credentials (username/password or token)
        DOCKERHUB_CREDS = credentials('dockerhub-creds')

        // Docker image name
        DOCKER_IMAGE = "vimalmahalingam/ci-cd-pipeline"
    }

    triggers {
        // Trigger build on every GitHub push
        githubPush()
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
                sh 'python3 -m unittest test_app.py'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t $DOCKER_IMAGE:${env.BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                sh "echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin"
                sh "docker push $DOCKER_IMAGE:${env.BUILD_NUMBER}"
            }
        }

        stage('Deploy') {
            steps {
                sh "docker run -d -p 8080:8080 $DOCKER_IMAGE:${env.BUILD_NUMBER}"
            }
        }
    }
}
