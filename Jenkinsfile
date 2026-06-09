pipeline {
    agent any

    environment {
        IMAGE_NAME = 'hencohen888/flask-aws-monitor'
    }

    stages {

        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Parallel Checks') {
            parallel {

                stage('Linting') {
                    steps {
                        sh 'python3 -m pip install --break-system-packages flake8'
                        sh 'python3 -m flake8 app/ --ignore=E501'
                    }
                }

                stage('Security Scan') {
                    steps {
                        sh 'python3 -m pip install --break-system-packages bandit'
                        sh 'python3 -m bandit -r app/'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -f docker/Dockerfile -t ${IMAGE_NAME}:${BUILD_NUMBER} -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKERHUB_USER',
                    passwordVariable: 'DOCKERHUB_PASS'
                )]) {
                    sh 'echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin'
                    sh 'docker push ${IMAGE_NAME}:${BUILD_NUMBER}'
                    sh 'docker push ${IMAGE_NAME}:latest'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed! Check logs for details.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}