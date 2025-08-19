pipeline {
  agent any
  environment {
    REGISTRY = "192.168.64.16:5000"     // registryVM IP:port
    IMAGE = "${REGISTRY}/helloworld"
  }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') {
      steps {
        script {
          SHORT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
          sh """
            echo ">>> Building ${IMAGE}:${SHORT_SHA}"
            docker build -t ${IMAGE}:${SHORT_SHA} .
          """
        }
      }
    }
    stage('Push') {
      steps {
        sh """
          docker push ${IMAGE}:${SHORT_SHA}
          docker tag ${IMAGE}:${SHORT_SHA} ${IMAGE}:latest || true
          docker push ${IMAGE}:latest || true
        """
      }
    }
  }
  post { success { echo "Pushed: ${IMAGE}:${SHORT_SHA}" } }
}
