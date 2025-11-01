pipeline {
  agent any

  environment {
    # ---- GHCR Settings ----
    GHCR_USERNAME = "hikiuzrx"
    GHCR_TOKEN = "ghp_yourClassicTokenHere"    // 👈 replace with your GHCR classic token

    # ---- Image Tags ----
    IMAGE_BACKEND = "ghcr.io/hikiuzrx/contract-backend:latest"
    IMAGE_FRONTEND = "ghcr.io/hikiuzrx/contract-frontend:latest"

    # ---- ECS Server ----
    ECS_HOST = "150.40.161.66"
    SSH_USER = "root"                          // 👈 change if not root
    SSH_PASS = "Zynfy1-rerkor-nibjid"          // 👈 replace with ECS password
  }

  stages {

    stage('Login to GHCR') {
      steps {
        sh '''
          echo "🔐 Logging in to GHCR..."
          echo $GHCR_TOKEN | docker login ghcr.io -u $GHCR_USERNAME --password-stdin
        '''
      }
    }

    stage('Build & Push Backend Image') {
      steps {
        sh '''
          echo "🚀 Building backend image..."
          docker build -t $IMAGE_BACKEND -f backend/Dockerfile backend

          echo "📤 Pushing backend image to GHCR..."
          docker push $IMAGE_BACKEND
        '''
      }
    }

    stage('Build & Push Frontend Image') {
      steps {
        sh '''
          echo "🚀 Building frontend image..."
          docker build -t $IMAGE_FRONTEND -f frontend/Dockerfile frontend

          echo "📤 Pushing frontend image to GHCR..."
          docker push $IMAGE_FRONTEND
        '''
      }
    }

    stage('Deploy to ECS (Docker Swarm)') {
      steps {
        sh '''
          echo "🔗 Connecting to ECS and deploying stack..."
          sshpass -p "$SSH_PASS" ssh -o StrictHostKeyChecking=no $SSH_USER@$ECS_HOST "
            echo '📦 Pulling latest images...'
            docker pull $IMAGE_BACKEND
            docker pull $IMAGE_FRONTEND

            echo '📂 Updating stack file (contract-stack.yaml)...'
            cd ~
            if [ -f contract-stack.yaml ]; then
              echo '✔ Stack file found.'
            else
              echo '⚠️ Stack file not found. Please upload it to ~/ on ECS.'
            fi

            echo '🚀 Deploying stack...'
            docker stack deploy -c ~/contract-stack.yaml contract_stack

            echo '🧹 Cleaning unused images...'
            docker image prune -af || true

            echo '✅ Swarm deployment completed successfully!'
          "
        '''
      }
    }
  }

  post {
    success {
      echo "🎉 Deployment succeeded!"
    }
    failure {
      echo "💥 Deployment failed!"
    }
  }
}
