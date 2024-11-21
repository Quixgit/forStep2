pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'app_quix'  // Название вашего Docker образа
        DOCKER_USERNAME = 'quixq'  // Ваше имя пользователя на Docker Hub
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')  // Получаем Docker Hub креды
    }
    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm  // Проверка исходного кода из репозитория
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Строим Docker образ с тегом latest
                    sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:latest .'  // Указан правильный формат имени образа
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Запуск тестов в контейнере
                    sh 'docker run --rm $DOCKER_USERNAME/$DOCKER_IMAGE:latest npm run test'  // Тестирование образа
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
                            docker tag $DOCKER_USERNAME/$DOCKER_IMAGE:latest $DOCKER_USERNAME/forstep:latest
                            docker push $DOCKER_USERNAME/forstep:latest
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'  // Завершение пайплайна
        }
        success {
            echo 'Tests passed!'  // Сообщение при успешном выполнении тестов
        }
        failure {
            echo 'Tests failed'  // Сообщение при неудаче тестов
        }
    }
}
