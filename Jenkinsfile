pipeline {
    agent any

    environment {
        REGISTRY = 'sanjanadas'
        IMAGE_NAME = 'shopease'
        K8S_NAMESPACE = 'ecommerce'
        DEPLOYMENT_NAME = 'shopease'
        CONTAINER_NAME = 'shopease'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Read Version') {
            steps {
                script {
                    env.APP_VERSION = readFile('active-version.txt').trim()
                    env.IMAGE_TAG = "${env.APP_VERSION}-${env.BUILD_NUMBER}"
                    env.FULL_IMAGE = "${env.REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"

                    echo "App version: ${env.APP_VERSION}"
                    echo "Image: ${env.FULL_IMAGE}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${env.FULL_IMAGE} ./app/${env.APP_VERSION}
                """
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                        docker push ${env.FULL_IMAGE}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl set image deployment/${env.DEPLOYMENT_NAME} ${env.CONTAINER_NAME}=${env.FULL_IMAGE} -n ${env.K8S_NAMESPACE}
                    kubectl rollout status deployment/${env.DEPLOYMENT_NAME} -n ${env.K8S_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful: ${env.FULL_IMAGE}"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}

