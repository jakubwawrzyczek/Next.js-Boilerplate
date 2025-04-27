pipeline {
  agent any

  environment {
    IMAGE_NAME_BUILD   = "nextjs-app-build"
    IMAGE_NAME_TEST    = "nextjs-app-test"
    IMAGE_NAME_DEPLOY  = "nextjs-app-deploy"
    CONTAINER_NAME     = "nextjs-app"

    // versioned tag: YYYYMMDD-<build number>
    BUILD_DATE = new Date().format("yyyyMMdd")
    BUILD_TAG  = "${BUILD_DATE}-${env.BUILD_NUMBER}"
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
          # Build the production image
          docker build -f Dockerfile.deploy \
            -t ${IMAGE_NAME_DEPLOY}:${BUILD_TAG} .

          echo "🗑 Removing any old deploy containers by image…"
          docker ps -aq --filter ancestor=${IMAGE_NAME_DEPLOY} | xargs -r docker rm -f

          echo "🗑 Removing any old container named ${CONTAINER_NAME}…"
          docker rm -f ${CONTAINER_NAME} >/dev/null 2>&1 || true

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
        sh '''
          echo "🩺 Waiting 5 seconds for the application to start…"
          sleep 5

          echo "🔍 Running healthcheck using curlimages/curl…"
          docker run --rm \
            --network container:${CONTAINER_NAME} \
            curlimages/curl:latest \
            curl --fail --max-time 5 http://localhost:3000/ \
            || (echo "❌ Healthcheck failed: application did not respond" && exit 1)

          echo "✅ Healthcheck passed"
        '''
      }
    }

    stage('Publish') {
      steps {
        script {
          def fullTag = "${IMAGE_NAME_DEPLOY}:${BUILD_TAG}"
          echo "💾 Saving image to ${IMAGE_NAME_DEPLOY}-${BUILD_TAG}.tar"
          sh "docker save ${fullTag} -o ${IMAGE_NAME_DEPLOY}-${BUILD_TAG}.tar"
        }
        archiveArtifacts artifacts: "${IMAGE_NAME_DEPLOY}-${BUILD_TAG}.tar", fingerprint: true
        echo "✅ Published artifact: ${IMAGE_NAME_DEPLOY}-${BUILD_TAG}.tar"
      }
    }
  }

  post {
    always {
      sh 'docker system prune -f'
    }
  }
}
