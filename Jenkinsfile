pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds-muni')
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-muni')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key-muni')
    AWS_REGION = 'us-west-2'
    CLUSTER_NAME = 'munish-ecommerce-cluster'
    DOCKER_CONFIG = '/var/lib/jenkins/.docker' // ensure correct config path
  }

stages {
  stage('Install Buildx') {
     steps {
        sh '''
          export DOCKER_CLI_EXPERIMENTAL=enabled
          mkdir -p $DOCKER_CONFIG/cli-plugins
          curl -sSL https://github.com/docker/buildx/releases/latest/download/buildx-linux-amd64 \
            -o $DOCKER_CONFIG/cli-plugins/docker-buildx
          chmod +x $DOCKER_CONFIG/cli-plugins/docker-buildx
          export PATH=$DOCKER_CONFIG/cli-plugins:$PATH
          docker buildx version
        '''
      }
    }

    stage('Build Docker Images') {
      steps {
        script {
          def services = ['user-service', 'product-service', 'cart-service', 'order-service']
          for (svc in services) {
            sh """
              export PATH=$DOCKER_CONFIG/cli-plugins:\$PATH
              docker buildx create --use --name mybuilder || true
              docker buildx inspect mybuilder --bootstrap
              docker buildx build --load -t $DOCKERHUB_CREDENTIALS_USR/${svc}:latest ./backend/${svc}
            """
          }
          // Build frontend
          sh """
            export PATH=$DOCKER_CONFIG/cli-plugins:\$PATH
            docker buildx build --load -t $DOCKERHUB_CREDENTIALS_USR/frontend:latest ./frontend
          """
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
          def services = ['user-service', 'product-service', 'cart-service', 'order-service', 'frontend']
          for (svc in services) {
		sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
		sh "docker push $DOCKERHUB_CREDENTIALS_USR/${svc}:latest"
          }
	sh "docker push $DOCKERHUB_CREDENTIALS_USR/frontend:latest"
        }
      }
    }

    stage('Provision EKS Cluster') {
      steps {
        sh '''
        eksctl create cluster \
          --name $CLUSTER_NAME \
          --region $AWS_REGION \
          --nodegroup-name standard-workers \
          --node-type t3.medium \
          --nodes 2
        '''
      }
    }

    stage('Configure kubeconfig') {
      steps {
        sh '''
        aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME
        '''
      }
    }

    stage('Deploy Backend Services to EKS') {
      steps {
        sh '''
        kubectl apply -f k8s/user-deployment.yaml
        kubectl apply -f k8s/user-service.yaml
        kubectl apply -f k8s/product-service.yaml
        kubectl apply -f k8s/cart-service.yaml
        kubectl apply -f k8s/order-service.yaml
        '''
      }
    }

    stage('Deploy Ingress for Backend') {
      steps {
        sh '''
        kubectl apply -f k8s/backend-ingress.yaml
        '''
      }
    }

    stage('Deploy Frontend') {
      steps {
        sh '''
        kubectl apply -f k8s/frontend-deployment.yaml
        kubectl apply -f k8s/frontend-service.yaml
        kubectl apply -f k8s/frontend-ingress.yaml
        '''
      }
    }
  }

  post {
    failure {
      echo 'Build Failed!'
    }
    success {
      echo 'Pipeline completed successfully!'
    }
  }
}
