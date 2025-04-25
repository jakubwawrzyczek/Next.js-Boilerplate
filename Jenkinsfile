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
                    rm -f .env.local
                      cp "$ENV_FILE" ./.env.local
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                script {
                sh """
                    docker build \
                    -f Dockerfile.build \
                    -t ${IMAGE_NAME_BUILD}:${BUILD_TAG} \
                    -t ${IMAGE_NAME_BUILD}:latest \
                    .
                """
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

    stage('Healthcheck') {
        steps {
            script {
            def cid = sh(
                script: "docker ps -q --filter ancestor=${IMAGE_NAME_DEPLOY}:${BUILD_TAG}",
                returnStdout: true
            ).trim()


            sh """
                echo "Checking HTTP response from container ${cid}..."
                docker exec ${cid} curl -f --max-time 5 http://localhost:3000/ || (
                echo "❌ Healthcheck failed!" && exit 1
                )
                echo "✅ Healthcheck passed"
            """
            }
        }
    }



    post {
        always {
            sh 'docker system prune -f'
        }
    }
}
