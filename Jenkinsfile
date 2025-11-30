pipeline {
  agent any

  environment {
    REGISTRY = "image-registry.openshift-image-registry.svc:5000"
    PROJECT = "bala12121986-dev"
    IMAGE_NAME = "enterprise-app"
  }

  stages {
    
    stage('Checkout Code') {
      steps {
        git branch: 'main', url: 'https://github.com/balagcpcloud/enterprise-platform-gitops.git'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          docker build -t ${REGISTRY}/${PROJECT}/${IMAGE_NAME}:latest ./app
        """
      }
    }

    stage('Login to OpenShift Registry') {
      steps {
        withCredentials([string(credentialsId: 'OC_TOKEN', variable: 'TOKEN')]) {
          sh """
            docker login ${REGISTRY} -u openshift -p ${TOKEN}
          """
        }
      }
    }

    stage('Push Image to OpenShift') {
      steps {
        sh """
          docker push ${REGISTRY}/${PROJECT}/${IMAGE_NAME}:latest
        """
      }
    }

    stage('Update Helm values.yaml') {
      steps {
        sh """
          sed -i 's/tag:.*/tag: "latest"/' helm/enterprise-app/values.yaml
        """
      }
    }

    stage('Commit and Push Back to GitHub') {
      steps {
        withCredentials([string(credentialsId: 'GIT_TOKEN', variable: 'GTOKEN')]) {
          sh """
            git config user.email "balagcpcloud@gmail.com"
            git config user.name "Bala"

            git add helm/enterprise-app/values.yaml || true
            git commit -m "Updated image tag to latest" || true

            git push https://balagcpcloud:${GTOKEN}@github.com/balagcpcloud/enterprise-platform-gitops.git main
          """
        }
      }
    }
  }
}

