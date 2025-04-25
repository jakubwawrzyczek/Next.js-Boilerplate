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

            echo "🗑  Removing any old deploy containers…"
            # remove ALL containers (running or stopped) from this image
            docker ps -aq --filter ancestor=${IMAGE_NAME_DEPLOY} | xargs -r docker rm -f

            # (optional extra guard: remove any container binding port 3000)
            docker ps -q --filter publish=3000 | xargs -r docker rm -f

            echo "🚀 Starting fresh container ${CONTAINER_NAME}"
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
            echo "Checking HTTP response on host port 3000…"
            curl -f --max-time 5 http://localhost:3000/ || (
                echo "❌ Healthcheck failed!" && exit 1
            )
            echo "✅ Healthcheck passed"
            """
        }
    }

  post {
    always {
      sh 'docker system prune -f'
    }
  }
}
