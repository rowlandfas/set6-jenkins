pipeline {
    agent any
 
    options {
        timestamps()
        skipDefaultCheckout(true)
    }
 
    environment {
        IMAGE_NAME = 'local-cicd-demo'
        CONTAINER_NAME = 'local-cicd-demo'
        APP_PORT = '8081'
    }
 
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out application from GitHub'
                checkout scm
 
                sh '''
                    echo "Workspace contents:"
                    pwd
                    ls -la
                    ls -la app
                '''
            }
        }
 
        stage('Test') {
            steps {
                echo 'Testing application files'
 
                sh '''
                    set -eu
 
                    test -f app/index.html
                    test -f app/Dockerfile
                    grep -q "Welcome to Set33 CI/CD Pipeline" app/index.html
 
                    echo "All application checks passed."
                '''
            }
        }
 
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
 
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} app
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                '''
            }
        }
 
        stage('Deploy Application') {
            steps {
                echo 'Deploying application locally'
 
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
 
                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:80 \
                        ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
 
        stage('Verify Deployment') {
            steps {
                echo 'Verifying deployed application'
 
                sh '''
                    set -eu
 
                    for i in $(seq 1 15); do
                        if docker exec ${CONTAINER_NAME} \
                            wget -q -O - http://127.0.0.1/ \
                            | grep -q "Welcome to My First CI/CD Pipeline"; then
                            echo "Deployment successful."
                            exit 0
                        fi
 
                        sleep 2
                    done
 
                    echo "Deployment verification failed."
                    exit 1
                '''
            }
        }
    }
 
    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
            echo 'Application URL: http://localhost:8081'
        }
 
        failure {
            echo 'Pipeline failed. Review the stage logs.'
        }
    }
}
