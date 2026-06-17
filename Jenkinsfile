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

        stage('Update GitOps Repo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-credentials',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                        rm -rf devops-final-gitops

                        git clone https://${GIT_USER}:${GIT_TOKEN}@github.com/HenCohen888/devops-final-gitops.git

                        cd devops-final-gitops

                        sed -i "s/tag: .*/tag: \\"${BUILD_NUMBER}\\"/" flask-aws-monitor/dev/values.yaml

                        git config user.email "jenkins@local"
                        git config user.name "jenkins"

                        git add flask-aws-monitor/dev/values.yaml
                        git commit -m "Update dev image tag to ${BUILD_NUMBER}" || echo "No changes to commit"

                        git push origin main
                    '''
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