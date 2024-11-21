pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'quixq/forstep' // Указываем репозиторий и образ
        DOCKER_CREDENTIALS = credentials('dockerhub-creds') // Получаем Docker Hub креды
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
                    // Строим Docker образ с тегом на основе хеша коммита
                    sh 'docker build -t $DOCKER_IMAGE:${GIT_COMMIT} .'
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    // Запуск тестов в контейнере
                    sh 'docker run --rm $DOCKER_IMAGE:${GIT_COMMIT} npm run test'
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Логинимся в Docker Hub
                        sh """
                            echo \$DOCKER_PASSWORD | docker login --username \$DOCKER_USERNAME --password-stdin
                            docker push \$DOCKER_IMAGE:${GIT_COMMIT} // Пушим с тегом на основе коммита
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
