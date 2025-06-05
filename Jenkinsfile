pipeline {
  agent any

  environment {
    GCP_PROJECT = 'my-gcp-project'
    IMAGE_TAG = "${env.BUILD_NUMBER}"
    IMAGE_REPO = "us-central1-docker.pkg.dev/${GCP_PROJECT}/repo/ecommerce-app"
    IMAGE_NAME = "${IMAGE_REPO}:${IMAGE_TAG}"
    SONARQUBE_ENV = credentials('sonar-token')
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/your-org/ecommerce-devops-project.git'
      }
    }

    stage('SonarQube Code Scan') {
      steps {
        withSonarQubeEnv('MySonarQubeServer') {
          sh 'sonar-scanner -Dsonar.projectKey=ecommerce-app -Dsonar.sources=./app-code -Dsonar.login=$SONARQUBE_ENV'
        }
      }
    }

    stage('Unit Test') {
      steps {
        sh 'cd app-code && npm install && npm test'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME ./docker'
      }
    }

    stage('Push to GAR') {
      steps {
        sh '''
          gcloud auth configure-docker
          docker push $IMAGE_NAME
        '''
      }
    }

    stage('Update Helm values.yaml') {
      steps {
        sh """
          echo "Updating Helm values.yaml with image tag: $IMAGE_TAG"
          sed -i 's|tag:.*|tag: $IMAGE_TAG|' helm/ecommerce/values.yaml
          sed -i 's|repository:.*|repository: $IMAGE_REPO|' helm/ecommerce/values.yaml
        """
      }
    }

    stage('Deploy to GKE via Helm') {
      steps {
        sh '''
          helm upgrade --install ecommerce-app ./helm/ecommerce
        '''
      }
    }

    stage('Terraform Infra (Optional)') {
      steps {
        dir('infra') {
          sh '''
            terraform init
            terraform plan
            terraform apply -auto-approve
          '''
        }
      }
    }
  }
}
