pipeline {
    agent any
    tools { jdk 'jdk-17-local' }


    environment {
        DOCKERHUB_CREDENTIALS = 'docker-hub' // Your Jenkins Docker Hub credentials
        IMAGE_BASE = 'alimiheb/demo-app'
    }

    triggers {
        pollSCM('* * * * *') // Poll every minute
    }

    stages {
        stage('Build & Test with Maven') {
            steps {
                sh 'mvn clean install' // Build and run tests
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imgTag = "${env.BUILD_NUMBER}"
                    docker.build("${env.IMAGE_BASE}:${imgTag}")
                    sh "docker tag ${env.IMAGE_BASE}:${imgTag} ${env.IMAGE_BASE}:latest"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('', env.DOCKERHUB_CREDENTIALS) {
                        sh "docker push ${env.IMAGE_BASE}:${env.BUILD_NUMBER}"
                        sh "docker push ${env.IMAGE_BASE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    sh 'kubectl rollout status deployment/demo-app-deployment'
                }
            }
        }
    }

    post {
        success {
            echo 'Build, test, Docker build, and push completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
