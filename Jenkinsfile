pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }
    tools {
        maven "mvn"
    }

    environment {
        IMAGE_NAME = "ghaithsaidani/crud-spring"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage("Tests") {
            steps {
                sh "mvn clean test"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-lts') {
                    sh 'mvn sonar:sonar'
                }
            }

            post {
                success {
                    script {
                        timeout(time: 2, unit: 'MINUTES') {
                            def qualityGate = waitForQualityGate()
                            if (qualityGate.status != 'OK') {
                                error "SonarQube Quality Gate failed: ${qualityGate.status}"
                            } else {
                                echo "SonarQube analysis passed."
                            }
                        }
                    }
                }
                failure {
                    echo "SonarQube analysis failed during execution."
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
