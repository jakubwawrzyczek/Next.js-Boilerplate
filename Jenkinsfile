pipeline {
  agent any

  environment {
    IMAGE_NAME_BUILD   = "nextjs-app-build"
    IMAGE_NAME_TEST    = "nextjs-app-test"
    IMAGE_NAME_DEPLOY  = "nextjs-app-deploy"
    BUILD_TAG          = "${env.BUILD_NUMBER}"
    CONTAINER_NAME     = "nextjs-app"        // ← fixed name for deploy container
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
        sh """
          docker build \
            -f Dockerfile.build \
            -t ${IMAGE_NAME_BUILD}:${BUILD_TAG} \
            -t ${IMAGE_NAME_BUILD}:latest \
            .
        """
      }
    }

    stage('Test') {
      steps {
        sh """
          docker build -f Dockerfile.test \
            -t ${IMAGE_NAME_TEST}:${BUILD_TAG} .
          docker run --rm ${IMAGE_NAME_TEST}:${BUILD_TAG}
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          # build prod image
          docker build -f Dockerfile.deploy \
            -t ${IMAGE_NAME_DEPLOY}:${BUILD_TAG} .

          # stop+remove any old container (frees up port 3000)
          if docker ps -q -f name=${CONTAINER_NAME}; then
            docker rm -f ${CONTAINER_NAME}
          fi

          # run the new one
          docker run -d \
            --name ${CONTAINER_NAME} \
            -p 3000:3000 \
            ${IMAGE_NAME_DEPLOY}:${BUILD_TAG}
        """
      }
    }

    stage('Healthcheck') {
      steps {
        sh """
          echo "Checking HTTP response from container ${CONTAINER_NAME}..."
          docker exec ${CONTAINER_NAME} \
            curl -f --max-time 5 http://localhost:3000/ || (
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
