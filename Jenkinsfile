pipeline {
    agent any
    environment {
        IMAGE_NAME = 'flask-app'
        VERSION = "v1.${BUILD_NUMBER}"
        BLUE_PORT = '5006'
        GREEN_PORT = '5007'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/kiranraju6/kiran.git', branch: 'main'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME}:${VERSION} .'
            }
        }

        stage('Run Green Container') {
            steps {
                sh '''
                docker rm -f flask-green || true
                docker run -d --name flask-green -p ${GREEN_PORT}:5000 ${IMAGE_NAME}:${VERSION}
                '''
            }
        }

        stage('Health Check') {
            steps {
                echo 'Waiting for green environment to boot...'
                sh 'sleep 5' // simple wait; you could use curl health check here
                sh 'curl -f http://host.docker.internal:${GREEN_PORT}/ || (echo "Health check failed!" && exit 1)'
            }
        }

        stage('Switch Traffic to Green') {
            steps {
                sh '''
                echo "Switching traffic to GREEN..."
                docker rm -f flask-blue || true
                docker rename flask-green flask-blue
                docker stop flask-blue
                docker start flask-blue
                '''
            }
        }
    }

    post {
        always {
            echo 'Blue-Green deployment complete.'
        }
    }
}
