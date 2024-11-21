pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'app_quix'
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')  // Получаем Docker Hub креды
        DOCKER_USERNAME = 'quixq'  // Ваше имя пользователя на Docker Hub
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
                    // Строим Docker образ с тегом latest
                    sh 'docker build -t $DOCKER_IMAGE:latest .'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Запуск тестов в контейнере
                    sh 'docker run --rm $DOCKER_IMAGE:latest npm run test'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Получаем хэш коммита
                        def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                        // Логинимся в Docker Hub
                        sh """
                            echo \$DOCKER_PASSWORD | docker login --username \$DOCKER_USERNAME --password-stdin
                            docker tag $DOCKER_IMAGE:latest \$DOCKER_USERNAME/$DOCKER_IMAGE:\$commitHash
                            docker push \$DOCKER_USERNAME/$DOCKER_IMAGE:\$commitHash
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
