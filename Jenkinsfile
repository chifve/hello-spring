pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
    }

    environment {
        DOCKERHUB_REPO = 'chifve/hello-spring'
        CONTAINER_NAME = 'hello-app'
        APP_PORT = '8070'
    }

    stages {

        stage('Build JAR') {
            steps {
                echo '🔨 Compilation de l application...'
                sh 'mvn clean package'
            }
        }

        stage('Build & Push Docker') {
            steps {
                echo '🐳 Construction et publication de l image Docker...'
                script {
                    docker.withRegistry('', 'dockerhub') {
                        def app = docker.build("${DOCKERHUB_REPO}:latest")
                        app.push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l application...'
                sh """
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker pull ${DOCKERHUB_REPO}:latest
                    docker run -d \
                      --name ${CONTAINER_NAME} \
                      -p ${APP_PORT}:${APP_PORT} \
                      ${DOCKERHUB_REPO}:latest
                """
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
    }
}