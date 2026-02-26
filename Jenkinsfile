pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
      - cat
    tty: true
"""
    }
  }

  environment {
    PROJECT_ID = "k8s-test-486501"
    REGION     = "asia-northeast3"
    REPO       = "devops-repo"
    IMAGE      = "nginx-sample"
    TAG        = "${BUILD_NUMBER}"
  }

  stages {
    stage('Build & Push') {
      steps {
        container('kaniko') {
          sh '''
          TOKEN=$(curl -s -H "Metadata-Flavor: Google" \
            http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token \
            | sed -n 's/.*"access_token":"\\([^"]*\\)".*/\\1/p')

          mkdir -p /kaniko/.docker
          cat > /kaniko/.docker/config.json <<EOF
{
  "auths": {
    "${REGION}-docker.pkg.dev": {
      "username": "oauth2accesstoken",
      "password": "${TOKEN}"
    }
  }
}
EOF

          /kaniko/executor \
            --context `pwd` \
            --dockerfile Dockerfile \
            --destination ${REGION}-docker.pkg.dev/${PROJECT_ID}/${REPO}/${IMAGE}:${TAG}
          '''
        }
      }
    }
  }
}