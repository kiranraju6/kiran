pipeline {
  agent any

  environment {
    IMAGE_NAME = 'flask-app'
    VERSION    = "v1.${env.BUILD_NUMBER}"
    BLUE_PORT  = '5001'
    GREEN_PORT = '5002'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/kiranraju6/kiran.git', branch: 'main'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          // build returns a Docker image object
          def img = docker.build("${IMAGE_NAME}:${VERSION}")
          // Optional: push to remote registry
          // docker.withRegistry('https://registry.example.com', 'docker-creds') {
          //   img.push()
          // }
        }
      }
    }

    stage('Run Green Container') {
      steps {
        sh """
          docker rm -f flask-green || true
          docker run -d --name flask-green -p ${GREEN_PORT}:5000 ${IMAGE_NAME}:${VERSION}
        """
      }
    }

    stage('Health Check') {
      steps {
        echo 'Waiting for green environment to boot...'
        sh 'sleep 5'
        sh "curl -f http://localhost:${GREEN_PORT}/ || (echo 'Health check failed!' && exit 1)"
      }
    }

    stage('Switch Traffic to Green') {
      steps {
        sh """
          echo "Switching traffic to GREEN"
          docker rm -f flask-blue || true
          docker rename flask-green flask-blue
          docker stop flask-blue && docker start flask-blue
        """
      }
    }
  }

  post {
    always {
      echo "Blue-Green deployment completed: ${IMAGE_NAME}:${VERSION}"
    }
    failure {
      echo "❌ Deployment failed. Cleaning up."
      sh 'docker rm -f flask-green || true'
    }
  }
}
