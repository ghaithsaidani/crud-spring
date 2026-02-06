pipeline {
    agent any

    environment {
        IMAGE_NAME = "ghaithsaidani/crud-spring"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("Tests") {
            steps{
                script {
                    docker.image('maven:3.9.6-eclipse-temurin-17')
                            .inside('-v /var/run/docker.sock:/var/run/docker.sock') {
                                sh "mvn clean test"
                            }
                }
            }
        }

        stage("Build Jar") {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage("Build Docker Image") {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${TAG} .
                docker tag ${IMAGE_NAME}:${TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage("Push to Docker Hub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${TAG}
                    docker push ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage("Deploy") {
            steps {
                sh """
                ssh azureuser@20.39.242.179 '
                  cd /opt/apps/crud-app &&
                  sudo sed -i "s/BACKEND_TAG=.*/BACKEND_TAG=${TAG}/" .env &&
                  docker compose pull backend &&
                  docker compose up -d backend
                '
                """
            }
        }
    }
}
