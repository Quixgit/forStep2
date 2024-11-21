pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')
    }
    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t app_quix .'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Здесь явно указываем команду для тестирования
                    sh 'docker run --rm app_quix npm run test'
                }
            }
        }
        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                script {
                    sh 'docker push app_quix'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
        success {
            echo 'Tests passed!'
        }
        failure {
            echo 'Tests failed'
        }
    }
}
