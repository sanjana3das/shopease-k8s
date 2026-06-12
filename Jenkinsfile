pipeline {
    agent any

    environment {
        REGISTRY      = 'sanjanadas'
        IMAGE_NAME    = 'shopease'
        K8S_NAMESPACE = 'ecommerce'
        DEPLOY_NAME   = 'shopease'
        CONTAINER     = 'shopease'
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
                echo "Code checked out from GitHub"
            }
        }

        stage('Read Version') {
            steps {
                script {
                    env.APP_VERSION = readFile('active-version.txt').trim()
                    env.IMAGE_TAG   = "${env.APP_VERSION}-build-${env.BUILD_NUMBER}"
                    env.FULL_IMAGE  = "${env.REGISTRY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"

                    echo "Version  : ${env.APP_VERSION}"
                    echo "Image Tag: ${env.IMAGE_TAG}"
                    echo "Full Image: ${env.FULL_IMAGE}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${FULL_IMAGE} ./app/${APP_VERSION}"
                echo "Docker image built: ${FULL_IMAGE}"
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
                        docker push ${FULL_IMAGE}
                        docker logout
                    """
                }
                echo "Image pushed to Docker Hub"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl set image deployment/${DEPLOY_NAME} \
                    ${CONTAINER}=${FULL_IMAGE} \
                    -n ${K8S_NAMESPACE}

                    kubectl rollout status deployment/${DEPLOY_NAME} \
                    -n ${K8S_NAMESPACE} \
                    --timeout=120s
                """
                echo "Deployment successful - Zero Downtime Rolling Update Done"
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                    echo "Current Pods:"
                    kubectl get pods -n ${K8S_NAMESPACE}

                    echo "Current Image:"
                    kubectl get deployment ${DEPLOY_NAME} -n ${K8S_NAMESPACE} -o wide
                """
            }
        }
    }

    post {
        success {
            echo "BUILD SUCCESSFUL - ShopEase ${APP_VERSION} is LIVE"
        }
        failure {
            echo "BUILD FAILED - Check logs above"
        }
    }
}

