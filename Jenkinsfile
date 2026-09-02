pipeline {

    agent any

    environment {

        DOCKERHUB_USERNAME = 'gopinathbca35'

        FRONTEND_IMAGE = "${DOCKERHUB_USERNAME}/bezkoder-frontend"
        BACKEND_IMAGE  = "${DOCKERHUB_USERNAME}/bezkoder-backend"

        IMAGE_TAG = "${BUILD_NUMBER}"

        EC2_HOST = '15.252.167.160'
        EC2_USER = 'ubuntu'

        APP_DIR = '/home/ubuntu/docker-compose-react-nodejs-mysql'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'

              
            }
        }

        stage('Build Frontend Image') {
            steps {
                echo 'Building frontend Docker image...'

                sh '''
                    docker build \
                      --build-arg REACT_APP_API_BASE_URL=http://${EC2_HOST}:6868/api \
                      -t ${FRONTEND_IMAGE}:${IMAGE_TAG} \
                      -t ${FRONTEND_IMAGE}:latest \
                      ./bezkoder-ui
                '''
            }
        }

        stage('Build Backend Image') {
            steps {
                echo 'Building backend Docker image...'

                sh '''
                    docker build \
                      -t ${BACKEND_IMAGE}:${IMAGE_TAG} \
                      -t ${BACKEND_IMAGE}:latest \
                      ./bezkoder-api
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Push Images to Docker Hub') {
            steps {

                sh '''
                    docker push ${FRONTEND_IMAGE}:${IMAGE_TAG}
                    docker push ${FRONTEND_IMAGE}:latest

                    docker push ${BACKEND_IMAGE}:${IMAGE_TAG}
                    docker push ${BACKEND_IMAGE}:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {

                sh '''
                    ssh -o StrictHostKeyChecking=no \
                    ${EC2_USER}@${EC2_HOST} << EOF

                    set -e

                    cd ${APP_DIR}

                    echo "Pulling latest frontend image..."
                    docker pull ${FRONTEND_IMAGE}:latest

                    echo "Pulling latest backend image..."
                    docker pull ${BACKEND_IMAGE}:latest

                    echo "Deploying application..."
                    docker compose up -d

                    echo "Deployment completed."

                    EOF
                '''
            }
        }
    }

    post {

        success {
            echo 'CI/CD Pipeline completed successfully.'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Jenkins console logs.'
        }
    }
}
