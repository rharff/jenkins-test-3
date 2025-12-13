pipeline {
    agent {
        kubernetes {
            label 'kaniko-agent'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    args:
      - sleep
      - infinity
    volumeMounts:
      - name: docker-config
        mountPath: /kaniko/.docker
  volumes:
    - name: docker-config
      projected:
        sources:
          - secret:
              name: dockerhub-secret
              items:
                - key: .dockerconfigjson
                  path: config.json
"""
        }
    }

    environment {
        DOCKERHUB_USERNAME = "<your-dockerhub-username>"
        IMAGE_NAME = "nodejs-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE = "docker.io/${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Image (Kaniko)') {
            steps {
                container('kaniko') {
                    sh """
                    /kaniko/executor \
                      --context ${WORKSPACE} \
                      --dockerfile Dockerfile \
                      --destination ${IMAGE} \
                      --destination docker.io/${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest \
                      --force
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/nodejs-app \
                  nodejs-app=${IMAGE} \
                  -n default --record
                """
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD pipeline completed successfully"
        }
        failure {
            echo "❌ CI/CD pipeline failed"
        }
    }
}
