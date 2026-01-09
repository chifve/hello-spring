pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'chifve'
        APP_NAME = 'hello-spring'
        IMAGE_TAG = 'latest'
        CONTAINER_NAME = 'hello-app'
        DOCKER_CRED_ID = 'dockerhub1'
    }

    stages {

        stage('Build JAR') {
            steps {
                echo '🔨 Building the application...'
                sh 'chmod +x mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                echo '🐳 Building and pushing Docker image...'
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
                echo '🚀 Deploying the application...'
                sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker pull $DOCKERHUB_USERNAME/$APP_NAME:$IMAGE_TAG
                    docker run -d --name $CONTAINER_NAME -p 8080:8080 $DOCKERHUB_USERNAME/$APP_NAME:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline failed.'
        }
    }
}
