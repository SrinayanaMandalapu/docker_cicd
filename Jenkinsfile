pipeline{
    agent any
    environment{
        DOCKERHUB_CREDENTIALS=credentials('dockerhub')
        IMAGE_NAME='srinayana20/docker_cicd'
        GIT_REPO='https://github.com/deepthisharanya1615/dev2int'
        BRANCH='main'
    }

    stages{
        stage('Check Source'){
            steps{
                echo 'Cloning git'
                git branch: "${BRANCH}", url: "${GIT_REPO}"
            }
        }
        stage('Build docker image'){
            steps{
                echo 'Building'
                bat "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Login to Docker Hub'){
            steps{
                echo 'Logging'
                bat "docker login -u ${DOCKERHUB_CREDENTIALS_USR} -p ${DOCKERHUB_CREDENTIALS_PSW}"
            }
        }
        stage('Push Image to Docker Hub'){
            steps{
                echo 'Pushing image'
                bat "docker push ${IMAGE_NAME}"
            }
        }
    }
}