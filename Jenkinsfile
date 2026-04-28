Copy

pipeline {
    agent any
 
    environment {
        IMAGE_NAME = 'nodejs-project'
        CONTAINER_NAME = 'nodejs-project-app'
        APP_PORT = '5000'
    }
 
    stages {
 
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest'
            }
        }
 
        stage('Deploy') {
            steps {
                sh '''
                    # Stop and remove any existing container
                    if [ $(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                    fi
 
                    # Run the new container
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:${APP_PORT} \
                        ${IMAGE_NAME}:latest
                '''
            }
        }
 
        stage('Verify Deployment') {
            steps {
                sh 'docker ps -f name=${CONTAINER_NAME}'
            }
        }
    }
 
    post {
        success {
            echo 'Build and deployment successful. App is running on port ${APP_PORT}.'
        }
        failure {
            echo 'Pipeline failed - check logs above.'
            sh '''
                if [ $(docker ps -aq -f name=${CONTAINER_NAME}) ]; then
                    docker stop ${CONTAINER_NAME}
                    docker rm ${CONTAINER_NAME}
                fi
            '''
        }
        always {
            // Remove old images, keeping only the last 3 builds
            sh 'docker image prune -f'
        }
    }
}
