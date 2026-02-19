pipeline {
  agent any

  environment {
    PROJECT_ID = 'your-gcp-project-id'
    REGION = 'asia-northeast3'
    REPOSITORY = 'devops-repo'
    IMAGE_NAME = 'nginx-sample'

    // e.g. asia-northeast3-docker.pkg.dev
    REGISTRY_HOST = "${REGION}-docker.pkg.dev"
    IMAGE_URI = "${REGISTRY_HOST}/${PROJECT_ID}/${REPOSITORY}/${IMAGE_NAME}"
  }

  options {
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Image') {
      steps {
        script {
          sh 'docker build -t ${IMAGE_URI}:${BUILD_NUMBER} -t ${IMAGE_URI}:latest .'
        }
      }
    }

    stage('Auth to GCP') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key-json', variable: 'GCP_SA_KEY')]) {
          sh '''
            gcloud auth activate-service-account --key-file="$GCP_SA_KEY"
            gcloud config set project "$PROJECT_ID"
            gcloud auth configure-docker "$REGISTRY_HOST" --quiet
          '''
        }
      }
    }

    stage('Push Image') {
      steps {
        sh '''
          docker push ${IMAGE_URI}:${BUILD_NUMBER}
          docker push ${IMAGE_URI}:latest
        '''
      }
    }
  }

  post {
    success {
      echo "Pushed: ${IMAGE_URI}:${BUILD_NUMBER} and ${IMAGE_URI}:latest"
    }
  }
}
