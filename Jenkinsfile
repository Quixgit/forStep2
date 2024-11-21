pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'app_quix'
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')  // Получаем Docker Hub креды
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
                    sh 'sudo docker build -t $DOCKER_IMAGE .'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Запуск тестов в контейнере
                    sh 'docker run --rm $DOCKER_IMAGE npm run test'
                }
            }
        }
        stage('Push to Docker Hub') {
            when {
                branch 'main'  // push только если ветка main
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Логинимся в Docker Hub
                        sh """
                            echo \$DOCKER_PASSWORD | docker login --username \$DOCKER_USERNAME --password-stdin
                            docker push \$DOCKER_IMAGE
                        """
                    }
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
