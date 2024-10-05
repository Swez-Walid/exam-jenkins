pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials("DOCKER_HUB_PASS")
        DOCKER_ID = "swezwalid" // replace this with your docker-id
        DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Build images for movie and cast services
                    sh 'docker build -t $DOCKER_ID/movie_service:latest ./movie-service'
                    sh 'docker build -t $DOCKER_ID/cast_service:latest ./cast-service'
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
                docker push $DOCKER_ID/movie_service:latest
                docker push $DOCKER_ID/cast_service:latest
                '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Deploy each chart separately using Helm
                    sh 'helm upgrade --install cast-db ./cast-db --namespace dev'
                    sh 'helm upgrade --install movie-db ./movie-db --namespace dev'
                    sh 'helm upgrade --install cast-service ./cast-service --namespace dev'
                    sh 'helm upgrade --install movie-service ./movie-service --namespace dev'
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