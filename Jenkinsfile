pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'quixq/forstep' // Репозиторий и образ
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
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    sh "docker build -t ${DOCKER_IMAGE}:${commitHash} ."
                }
            }
        }
        stage('Run Tests') {
            steps {
                script {
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    // Запуск тестов в контейнере
                    sh "docker run --rm ${DOCKER_IMAGE}:${commitHash} npm run test"
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
                            docker push ${DOCKER_IMAGE}:${commitHash}
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
