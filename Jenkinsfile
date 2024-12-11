pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('docker-hub')
    }

    stages {
        stage('Build Docker Images') {
            steps {
                sh 'docker build -t $DOCKER_CREDENTIALS_USR/learner-frontend:1.0 ./frontend'
                sh 'docker build -t $DOCKER_CREDENTIALS_USR/learner-backend:1.0 ./backend'
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $DOCKER_CREDENTIALS_USR/learner-frontend:1.0'
                sh 'docker push $DOCKER_CREDENTIALS_USR/learner-backend:1.0'
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh 'helm upgrade --install learner-chart ./learner-chart'
            }
        }
    }
}
