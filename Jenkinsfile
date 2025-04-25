pipeline {
    agent any
    environment {
        IMAGE_NAME_BUILD = "nextjs-app-build"
        IMAGE_NAME_TEST  = "nextjs-app-test"
        IMAGE_NAME_DEPLOY = "nextjs-app-deploy"
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

stage('Prepare .env') {
            steps {
                withCredentials([file(credentialsId: 'env-local-file', variable: 'ENV_FILE')]) {
                    sh '''
                      cp "$ENV_FILE" ./.env.local
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    sh "docker build -f Dockerfile.build -t ${IMAGE_NAME_BUILD}:${BUILD_TAG} ."
                }
            }
        }
        stage('Test') {
            steps {
                script {
                sh """
                    docker build \
                    --build-arg BUILD_TAG=${BUILD_TAG} \
                    -f Dockerfile.test \
                    -t ${IMAGE_NAME_TEST}:${BUILD_TAG} \
                    .
                    docker run --rm ${IMAGE_NAME_TEST}:${BUILD_TAG}
                """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                sh """
                    docker build \
                    --build-arg BUILD_TAG=${BUILD_TAG} \
                    -f Dockerfile.deploy \
                    -t ${IMAGE_NAME_DEPLOY}:${BUILD_TAG} \
                    .
                    docker run -d -p 3000:3000 ${IMAGE_NAME_DEPLOY}:${BUILD_TAG}
                """
                }
            }
        }
    }
    post {
        always {
            sh 'docker system prune -f'
        }
    }
}
