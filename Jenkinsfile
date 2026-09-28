pipeline {
    agent any

    environment {
        IMAGE_NAME = 'gokulapriyam/kanban-dashboard'
        CONTAINER_NAME = 'kanban-app'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${IMAGE_NAME}:${BUILD_NUMBER}

                        docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        --memory=512m \
                        --cpus=0.5 \
                        -p 8080:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Validate Deployment') {
            steps {
                sh '''
                    sleep 15
                    curl -f http://127.0.0.1:8080/
                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
