pipeline {
    agent any

    environment {
        IMAGE_NAME = "vineeshraj/website"
        K8S_DEPLOYMENT = "website-deployment"
        K8S_CONTAINER = "website"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/rajvineesh-sudo/website.git'
            }
        }

        stage('Release Gate (25th only)') {
            steps {
                script {
                    def day = sh(script: "date +%d", returnStdout: true).trim()
                    if (day != "25") {
                        error("Release blocked. Allowed only on 25th, today is ${day}")
                    }
                    echo "Release gate passed (25th)"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                  docker build -t $IMAGE_NAME:$BUILD_NUMBER .
                  docker tag $IMAGE_NAME:$BUILD_NUMBER $IMAGE_NAME:latest
                '''
            }
        }

        stage('Docker Login (Token)') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-token',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_TOKEN'
                )]) {
                    sh '''
                      echo "$DOCKER_TOKEN" | docker login \
                      -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                sh '''
                  docker push $IMAGE_NAME:$BUILD_NUMBER
                  docker push $IMAGE_NAME:latest
                '''
            }
        }

        stage('Deploy to Kubernetes (Rolling Update)') {
            steps {
                sh '''
                  kubectl set image deployment/$K8S_DEPLOYMENT \
                  $K8S_CONTAINER=$IMAGE_NAME:$BUILD_NUMBER

                  kubectl rollout status deployment/$K8S_DEPLOYMENT
                '''
            }
        }

        stage('Cleanup Old Docker Images') {
            steps {
                sh '''
                  docker image prune -af --filter "until=168h"
                '''
            }
        }
    }

    post {
        success {
            echo "CI/CD pipeline completed successfully"
        }
        failure {
            echo "CI/CD pipeline failed"
        }
    }
}
