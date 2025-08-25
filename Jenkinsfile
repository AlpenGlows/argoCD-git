pipeline {
  agent any

  environment {
    REGISTRY = "192.168.64.16:5000"     // registry VM IP:port
    IMAGE    = "${REGISTRY}/helloworld" // sabit image adı
    REG_CRED = "regcred"                // (opsiyonel) Jenkins Credentials ID (username/password)
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    // (OPSİYONEL) Registry login gerekiyorsa bu aşamayı bırak
   stage('Docker Login') {
  when { expression { return env.REG_CRED?.trim() } }
  steps {
    withCredentials([usernamePassword(credentialsId: env.REG_CRED, usernameVariable: 'U', passwordVariable: 'P')]) {
      sh '''
        set -e
        echo "$P" | docker login "$REGISTRY" -u "$U" --password-stdin
      '''
    }
  }
}


    stage('Build') {
      steps {
        script {
          // >>> SADE DEĞİŞİKLİK #1: SHORT_SHA'yı env'e yaz (diğer stage'lerde de erişilsin)
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

          # İsteğe bağlı: 'latest' etiketini de güncelle
          docker tag  ${IMAGE}:${env.SHORT_SHA} ${IMAGE}:latest || true
          docker push ${IMAGE}:latest || true
        """
      }
    }
  }

  post {
    always {
      // (opsiyonel) login kullandıysan çıkış yap
      sh 'docker logout ' + env.REGISTRY + ' || true'
    }
    success { echo "Pushed: ${IMAGE}:${env.SHORT_SHA}" }
  }
}
