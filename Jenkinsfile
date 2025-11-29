pipeline {
    agent any

    environment {
        PYTHON = "C:\\Users\\manda\\AppData\\Local\\Programs\\Python\\Python312" 
        PATH = "${PYTHON};${PYTHON}\\Scripts;${env.PATH}"
        // Jenkins credentials for Docker Hub
        DOCKERHUB_CREDENTIALS = credentials('dockerhub')

        // Docker image name
        IMAGE_NAME  = 'srinayana20/docker_cicd'

        // Git repo details
        GIT_REPO    = 'https://github.com/SrinayanaMandalapu/docker_cicd'
        BRANCH      = 'main'

        // Kubernetes details
        K8S_NAMESPACE  = 'default'
        K8S_DEPLOYMENT = 'docker-cicd-deployment'
        K8S_CONTAINER  = 'docker-cicd-container'

        // Path to kubeconfig (adjust for your Jenkins node)
        KUBECONFIG = 'C:\\Users\\manda\\.kube\\config'
    }

    stages {
        stage('Checkout Source') {
            steps {
                echo 'Cloning Git repository...'
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Install Dependencies (Build)') {
            steps {
                echo 'Installing Python dependencies using pip...'
                // For Linux agents, use 'sh "pip install -r requirements.txt"'
                bat "pip install -r requirements.txt"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${BUILD_NUMBER}"
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

                    // 1) Create/Update deployment + service from YAML
                    bat "kubectl apply -f deployment.yaml -n ${K8S_NAMESPACE} --validate=false"

                    // 2) Update deployment image to the new tag
                    echo "Updating image in Kubernetes deployment..."
                    bat """
                        kubectl set image deployment/${K8S_DEPLOYMENT} \
                        ${K8S_CONTAINER}=${IMAGE_NAME}:${BUILD_NUMBER} \
                        -n ${K8S_NAMESPACE}
                    """

                    // 3) Wait until rollout succeeds
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
