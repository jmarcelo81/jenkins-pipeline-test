pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: kaniko
            image: gcr.io/kaniko-project/executor:v1.20.0-debug
            command:
            - sleep
            args:
            - 9999999
            volumeMounts:
            - name: docker-config
              mountPath: /kaniko/.docker
          volumes:
          - name: docker-config
            projected:
              sources:
              - secret:
                  name: harbor-dockerconfig
                  items:
                  - key: .dockerconfigjson
                    path: config.json
      '''
    }
  }

  environment {
    HARBOR_REGISTRY = 'harbor.jmarcelocarvalho.com'
    HARBOR_PROJECT  = 'library'
    IMAGE_NAME      = 'jenkins-pipeline-test'
    IMAGE_TAG       = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Build and Push') {
      steps {
        container('kaniko') {
          sh '''
            /kaniko/executor \
              --context=`pwd` \
              --dockerfile=Dockerfile \
              --destination=${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG} \
              --destination=${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:latest
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Image pushed: ${HARBOR_REGISTRY}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    failure {
      echo "❌ Pipeline failed"
    }
  }
}
