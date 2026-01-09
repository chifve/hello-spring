pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKERHUB_USERNAME = 'chifve'
        APP_NAME = 'hello-spring'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'hello-app'
        DOCKER_CRED_ID = 'dockerhub1'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/chifve/hello-spring.git',
                    credentialsId: DOCKER_CRED_ID
            }
        }

        stage('Build JAR') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: DOCKER_CRED_ID,
                            usernameVariable: 'USER',
                            passwordVariable: 'PASS'
                        )
                    ]) {
                        sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                        sh 'docker build -t $USER/$APP_NAME:$IMAGE_TAG .'
                        sh 'docker push $USER/$APP_NAME:$IMAGE_TAG'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop hello-app || true
                    docker rm hello-app || true
                    docker run -d --name hello-app -p 8080:8080 chifve/hello-spring:latest
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished'
        }
        success {
            echo '✅ Pipeline succeeded'
        }
        failure {
            echo '❌ Pipeline failed'
        }
    }
}
