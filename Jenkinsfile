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
      withCredentials([string(credentialsId: 'kubeconfig-text', variable: 'KUBECONFIG_CONTENT')]) {
        sh """
          set -e

          # kubeconfig'i dosyaya yaz
          KCONF=\$(mktemp)
          printf "%s" "\$KUBECONFIG_CONTENT" > "\$KCONF"

          # Değerleri hazırla
          IMG_REPO="${REGISTRY}/helloworld"
          IMG_TAG="${SHORT_SHA}"

          # Geçici render dosyası
          RENDERED=\$(mktemp)

          # Helm kullanmadan basit render: sadece image.repository ve image.tag'i değiştiriyoruz
          cp ./charts/helloworld/templates/deployment.yaml "\$RENDERED"
          sed -i 's#{{[ ]*\\.Values\\.image\\.repository[ ]*}}#'"$IMG_REPO"'#g' "\$RENDERED"
          sed -i 's#{{[ ]*\\.Values\\.image\\.tag[ ]*}}#'"$IMG_TAG"'#g' "\$RENDERED"

          echo '>>> Applying rendered deployment manifest'
          kubectl --kubeconfig="\$KCONF" -n default apply -f "\$RENDERED"

          echo '>>> Waiting rollout'
          kubectl --kubeconfig="\$KCONF" -n default rollout status deploy/helloworld

          # (İsteğe bağlı) Service varsa uygula
          if [ -f ./charts/helloworld/templates/service.yaml ]; then
            # service.yaml templating içeriyorsa benzer sed işlemi gerekebilir
            kubectl --kubeconfig="\$KCONF" -n default apply -f ./charts/helloworld/templates/service.yaml || true
          fi
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
