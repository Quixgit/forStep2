pipeline {
    agent none  // Пайплайн не будет выполняться на главном сервере Jenkins, только на worker-ах
    
    environment {
        DOCKER_USERNAME = 'quixq'  // Укажите свой Docker Hub логин
        DOCKER_PASSWORD = credentials('dockerhub-creds')  // Идентификатор кредов для Docker Hub в Jenkins
        DOCKER_IMAGE = 'app_quix'  // Название вашего Docker образа
    }

    stages {
        stage('Checkout') {
            agent any  // Этот этап будет выполнен на любом доступном агенте Jenkins
            steps {
                checkout scm  // Получаем исходный код из репозитория GitHub
            }
        }

        stage('Build Docker Image') {
            agent { label 'jenks2' }  // Указываем, что этот этап должен выполняться на worker-узле
            steps {
                script {
                    // Строим Docker образ с тегом latest
                    sh 'docker build -t $DOCKER_USERNAME/$DOCKER_IMAGE:latest .'
                }
            }
        }

        stage('Run Tests') {
            agent { label 'jenks2' }  // Тестирование также будет выполнено на worker-узле
            steps {
                script {
                    // Запускаем контейнер и выполняем тесты
                    def result = sh(script: 'docker run --rm $DOCKER_USERNAME/$DOCKER_IMAGE:latest npm run test', returnStatus: true)
                    // Если тесты не прошли (код возврата не 0), завершить пайплайн с ошибкой
                    if (result != 0) {
                        error "Tests failed"
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            agent { label 'jenks2' }  // Push образа в Docker Hub выполняется также на worker-узле
            steps {
                script {
                    // Логинимся в Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
                    }
                    
                    // Тегируем и пушим образ в Docker Hub
                    sh 'docker tag $DOCKER_USERNAME/$DOCKER_IMAGE:latest $DOCKER_USERNAME/$DOCKER_IMAGE:$BUILD_NUMBER'
                    sh 'docker push $DOCKER_USERNAME/$DOCKER_IMAGE:$BUILD_NUMBER'
                }
            }
        }
    }

    post {
        success {
            echo 'Docker image pushed to Docker Hub successfully!'
        }
        failure {
            echo 'Tests failed'
        }
    }
}
