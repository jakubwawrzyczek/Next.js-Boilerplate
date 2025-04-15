pipeline {
    agent any
    environment {
        DOCKER_HOST = "unix:///var/run/docker.sock"
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
                    sh "docker build -f Dockerfile.test -t ${IMAGE_NAME_TEST}:${BUILD_TAG} ."
                    sh "docker run --rm ${IMAGE_NAME_TEST}:${BUILD_TAG}"
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    sh "docker build -f Dockerfile.deploy -t ${IMAGE_NAME_DEPLOY}:${BUILD_TAG} ."
                    sh "docker run -d -p 3000:3000 ${IMAGE_NAME_DEPLOY}:${BUILD_TAG}"
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
