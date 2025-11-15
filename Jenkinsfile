pipeline {
    agent any
    tools { jdk 'jdk-17-local' }


    environment {
        DOCKERHUB_CREDENTIALS = 'docker-hub'
        IMAGE_BASE = 'alimiheb/demo-app'
        HELM_CHART_PATH = './demo-app-chart'
    }

    triggers {
        pollSCM('* * * * *') // Poll every minute
    }

    stages {
        stage('Build & Test with Maven') {
            steps {
                sh 'mvn clean install'
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

        stage('Deploy with Helm') {
            steps {
                withCredentials([file(credentialsId: '77b07b6e-8fcf-4199-9d27-5c34139ded93', variable: 'KUBECONFIG')]) {
                    sh 'helm upgrade --install demo-app ${HELM_CHART_PATH} --set image.tag=latest'
                }
            }
        }
    }

    post {
        success {
            echo 'Build, test, Docker build, push, and Helm deployment completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
