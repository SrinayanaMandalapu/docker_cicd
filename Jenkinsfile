pipeline {
    agent any

    environment {
        // Jenkins credentials ID for your Docker Hub username/password
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')

        // Docker image (no tag – we'll append :BUILD_NUMBER or :latest)
        IMAGE_NAME  = 'srinayana20/docker_cicd'

        // Git repo details
        GIT_REPO    = 'https://github.com/SrinayanaMandalapu/docker_cicd'
        BRANCH      = 'main'

        // Kubernetes details
        K8S_NAMESPACE  = 'default'
        K8S_MANIFEST   = 'deployment.yaml'     // path in your repo
        K8S_DEPLOYMENT = 'docker-cicd-deployment'  // must match metadata.name in Deployment
        K8S_CONTAINER  = 'docker-cicd-container'   // must match container name in Deployment
    }

    stages {

        stage('Checkout Source') {
            steps {
                echo 'Cloning Git repository...'
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${BUILD_NUMBER}"
                    // Build with a unique tag for each build
                    bat "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                echo 'Logging in to Docker Hub...'
                bat "docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}"
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    echo "Pushing image: ${IMAGE_NAME}:${BUILD_NUMBER}"
                    bat "docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Applying Kubernetes manifests..."
                    // Ensure deployment & service exist (create/update)
                    bat "kubectl apply -f ${K8S_MANIFEST} -n ${K8S_NAMESPACE}"

                    echo "Updating deployment image to ${IMAGE_NAME}:${BUILD_NUMBER}..."
                    // Update the image in the existing deployment
                    bat """
                        kubectl set image deployment/${K8S_DEPLOYMENT} \
                        ${K8S_CONTAINER}=${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n ${K8S_NAMESPACE}
                    """

                    echo "Waiting for rollout to complete..."
                    bat "kubectl rollout status deployment/${K8S_DEPLOYMENT} -n ${K8S_NAMESPACE}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully. Image deployed to Kubernetes."
        }
        failure {
            echo "Pipeline failed. Check the logs for details."
        }
    }
}
