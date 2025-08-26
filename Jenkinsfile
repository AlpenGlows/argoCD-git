pipeline {
  agent { label 'docker' }

  environment {
    REGISTRY = "192.168.64.16:5000"
    IMAGE    = "${REGISTRY}/helloworld"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        // veya: checkout([...]) // shallow=false önerisi yukarıda
      }
    }

    stage('Docker Login (opsiyonel)') {
      when { expression { return env.NEED_LOGIN == 'true' } }
      steps {
        withCredentials([usernamePassword(credentialsId: 'REGISTRY_CREDS', passwordVariable: 'REG_PASS', usernameVariable: 'REG_USER')]) {
          sh 'echo "$REG_PASS" | docker login "$REGISTRY" -u "$REG_USER" --password-stdin'
        }
      }
    }

    stage('Build') {
      steps {
        script {
          env.SHORT_SHA = sh(returnStdout: true, script: "git rev-parse --short=7 HEAD").trim()
        }
        sh '''
          echo ">>> Building ${IMAGE}:${SHORT_SHA}"
          docker build -t ${IMAGE}:${SHORT_SHA} .
        '''
      }
    }

    stage('Push') {
      steps {
        sh '''
          set -e
          echo ">>> Pushing ${IMAGE}:${SHORT_SHA}"
          docker push ${IMAGE}:${SHORT_SHA}
          docker tag  ${IMAGE}:${SHORT_SHA} ${IMAGE}:latest || true
          docker push ${IMAGE}:latest || true
        '''
      }
    }
  }

  post {
    success {
      echo "Pushed: ${IMAGE}:${env.SHORT_SHA}"
    }
    failure {
      sh 'docker logout ${REGISTRY} || true'
    }
    always {
      sh 'docker images | head -n 20 || true'
    }
  }
}
