pipeline {
  agent any

  environment {
    REGISTRY = "192.168.64.16:5000"     
    IMAGE    = "${REGISTRY}/helloworld"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Build') {
      steps {
        script {
          env.SHORT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
          sh """
            echo ">>> Building ${IMAGE}:${env.SHORT_SHA}"
            docker build -t ${IMAGE}:${env.SHORT_SHA} .
          """
        }
      }
    }

    stage('Push') {
      steps {
        sh """
          echo ">>> Pushing ${IMAGE}:${env.SHORT_SHA}"
          docker push ${IMAGE}:${env.SHORT_SHA}

          docker tag  ${IMAGE}:${env.SHORT_SHA} ${IMAGE}:latest || true
          docker push ${IMAGE}:latest || true
        """
      }
    }

    stage('Deploy') {
      steps {
        // Jenkins’te cluster’a erişmek için bir kubeconfig credential eklemiş olman lazım
        withKubeConfig(credentialsId: 'kubeconfig-id') {
          sh """
            helm upgrade --install helloworld ./charts/helloworld \
              -n default --create-namespace \
              --set image.repository=${REGISTRY}/helloworld \
              --set image.tag=${env.SHORT_SHA}

            kubectl -n default rollout status deploy/helloworld
          """
        }
      }
    }
  }

  post {
    success {
      echo "Pushed and deployed: ${IMAGE}:${env.SHORT_SHA}"
    }
  }
}
