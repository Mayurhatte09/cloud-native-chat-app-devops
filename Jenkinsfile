pipeline {
  agent any

  environment {
    DOCKER_HUB = "mayrhatte09"
    IMAGE_BACKEND = "chatapp-backend"
    IMAGE_FRONTEND = "chatapp-frontend"
    SONAR_HOME = tool "sonarqube"
  }

  stages {

    stage('Clean Workspace') {
      steps {
        cleanWs()
      }
    }

    stage('Checkout Code') {
      steps {
        git branch: 'main',
        url: 'https://github.com/mayurhatte09/cloud-native-chat-app.git'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("sonarqube") {
          sh """
          $SONAR_HOME/bin/sonar-scanner \
          -Dsonar.projectName=chat-app \
          -Dsonar.projectKey=chat-app
          """
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        sh """
        docker build -t $DOCKER_HUB/$IMAGE_BACKEND:latest ./backend
        docker build -t $DOCKER_HUB/$IMAGE_FRONTEND:latest ./frontend
        """
      }
    }

    stage('Push Docker Images') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'dockerhub-credentials',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {

          sh """
          echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
          docker push $DOCKER_HUB/$IMAGE_BACKEND:latest
          docker push $DOCKER_HUB/$IMAGE_FRONTEND:latest
          """
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh """
        kubectl apply -f kubernetes/
        """
      }
    }

  }

  post {
    success {
      echo " Pipeline executed successfully!"
    }
    failure {
      echo " Pipeline failed. Check logs."
    }
  }
}