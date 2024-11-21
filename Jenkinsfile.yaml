pipeline {
    agent {
        label 'jenks2'  // Указывает на агент Jenkins с меткой 'jenks2'
    }
    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-creds')  // Использует Jenkins Credentials для Docker
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                    git checkout "${BRANCH_NAME}"  // Клонирует указанный бранч из репозитория
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    // Строим Docker образ с тегом 'app_quix'
                    sh 'docker build -t app_quix .'  
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Запускаем Docker контейнер для выполнения тестов
                    sh 'docker run --rm app_quix npm test || exit 1'  
                }
            }
        }
        stage('Push to Docker Hub') {
            when {
                expression {
                    return currentBuild.result == null || currentBuild.result == 'SUCCESS'  // Только если предыдущие стадии успешны
                }
            }
            steps {
                script {
                    // Логинимся в Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                            docker tag app_quix quixq/quix/test-node-app:latest
                            docker push quixq/quix/test-node-app:latest
                        '''
                    }
                }
            }
        }
    }
    post {
        failure {
            echo 'Tests failed'  // Выводит сообщение, если тесты не прошли
        }
    }
}
