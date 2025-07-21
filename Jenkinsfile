pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds') // Jenkins credentials
    AWS_ACCESS_KEY_ID = credentials('aws-access-key')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    AWS_REGION = 'us-west-2'
    CLUSTER_NAME = 'munish-ecommerce-cluster'
  }

  stages {

    stage('Build Docker Images') {
      steps {
        script {
          def services = ['user-service', 'product-service', 'cart-service', 'order-service', 'frontend']
          for (svc in services) {
            sh "docker build -t $DOCKERHUB_CREDENTIALS_USR/${svc}:latest ./${svc}"
          }
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        script {
          sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
          def services = ['user-service', 'product-service', 'cart-service', 'order-service', 'frontend']
          for (svc in services) {
            sh "docker push $DOCKERHUB_CREDENTIALS_USR/${svc}:latest"
          }
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

    stage('Deploy to EKS') {
      steps {
        sh '''
        aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
        kubectl apply -f k8s/
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
