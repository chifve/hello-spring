pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'chifve'
        APP_NAME = 'hello-spring'

        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'hello-app'

        DOCKER_CRED_ID = 'dockerhub'
    }

    stages {
        stage('Build JAR') {
            steps {
                echo '🔨 Compilation de l\'application...'
                bat 'mvnw.cmd clean package -DskipTests'
            }
        }

        stage('Build & Push Docker') {
            steps {
                echo '🐳 Construction et publication de l\'image Docker...'
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKER_CRED_ID, usernameVariable: 'USER', passwordVariable: 'PASS')]) {

                        bat 'docker login -u %USER% -p %PASS%'

                        bat "docker build -t %USER%/%APP_NAME%:%IMAGE_TAG% ."
                        bat "docker push %USER%/%APP_NAME%:%IMAGE_TAG%"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l\'application...'
                bat """
                    docker stop ${CONTAINER_NAME} || exit 0
                    docker rm ${CONTAINER_NAME} || exit 0
                    docker pull ${DOCKERHUB_USERNAME}/${APP_NAME}:${IMAGE_TAG}
                    docker run -d --name ${CONTAINER_NAME} -p 8080:8080 ${DOCKERHUB_USERNAME}/${APP_NAME}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            bat 'docker logout'
        }
        success {
            echo '✅ Pipeline exécuté avec succès!'
        }
        failure {
            echo '❌ Le pipeline a échoué.'
        }
    }
}