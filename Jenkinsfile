pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'swezwalid'
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials-id')
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Build images for movie and cast services
                    sh 'docker build -t $DOCKER_HUB_REPO/movie_service:latest ./movie-service'
                    sh 'docker build -t $DOCKER_HUB_REPO/cast_service:latest ./cast-service'
                }
            }
        }

        stage('Push Docker Images to DockerHub') {
             environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }
            steps {
                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_HUB_REPO/movie_service:latest
                docker push $DOCKER_HUB_REPO/cast_service:latest
                '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy each chart separately using Helm
                    sh 'helm upgrade --install cast-db ./helm-charts/cast-db --namespace dev'
                    sh 'helm upgrade --install movie-db ./helm-charts/movie-db --namespace dev'
                    sh 'helm upgrade --install cast-service ./helm-charts/cast-service --namespace dev'
                    sh 'helm upgrade --install movie-service ./helm-charts/movie-service --namespace dev'
                }
            }
        }
    }

    post {
        always {
            // Clean up local Docker images
            sh 'docker rmi $DOCKER_HUB_REPO/movie_service:latest'
            sh 'docker rmi $DOCKER_HUB_REPO/cast_service:latest'
        }
    }
}