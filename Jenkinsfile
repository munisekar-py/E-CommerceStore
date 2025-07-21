pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds-muni')
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-muni')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key-muni')
    AWS_REGION = 'us-west-2'
    CLUSTER_NAME = 'munish-ecommerce-cluster'
  }

stages {

stage('Install Buildx') {
      steps {
        sh '''
          mkdir -p ~/.docker/cli-plugins
          curl -sSL https://github.com/docker/buildx/releases/latest/download/buildx-linux-amd64 \
            -o ~/.docker/cli-plugins/docker-buildx
          chmod +x ~/.docker/cli-plugins/docker-buildx

          docker buildx version
        '''
      }
}

    stage('Build Docker Images') {
      steps {
        script {
          def services = ['user-service', 'product-service', 'cart-service', 'order-service', 'frontend']
          for (svc in services) {
		sh '''
		docker buildx create --name mybuilder --driver docker-container || true
          	docker buildx inspect mybuilder --bootstrap
          	docker buildx build --load -t $DOCKERHUB_CREDENTIALS_USR/${svc}:latest ./backend/${svc}
        	'''            
          }
	sh "docker buildx build --load -t $DOCKERHUB_CREDENTIALS_USR/${svc}:latest ./frontend/${svc}"
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
