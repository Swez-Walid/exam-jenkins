pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "swezwalid" // replace this with your docker-id
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
DOCKER_PASS = credentials("DOCKER_HUB_PASS") 
}
agent any // Jenkins will be able to select all available agents
stages {
        stage('Build Docker Images') {
            steps {
                script {
                    // Construire l'image Docker pour le service movie_service
                    
                    sh 'docker build -t $DOCKER_ID/movie-service:${DOCKER_TAG} ./movie-app'

                    // Construire l'image Docker pour le service cast_service
                    sh 'docker build -t $DOCKER_ID/cast-service:${DOCKER_TAG} ./cast-app'
                }
            }
        }

        stage('Run Movie Database') {
            steps {
                script {
                    // Démarrer le conteneur de movie_db
                    sh '''
                    docker run --name movie_db -d \
                    -e POSTGRES_USER=movie_db_username \
                    -e POSTGRES_PASSWORD=movie_db_password \
                    -e POSTGRES_DB=movie_db_dev \
                    -v postgres_data_movie:/var/lib/postgresql/data/ \
                    postgres:12.1-alpine
                    '''

                    // Attendre que la base de données soit prête
                    sleep(15)  // Ajustez ce délai si nécessaire
                }
            }
        }

        stage('Run Cast Database') {
            steps {
                script {
                    // Démarrer le conteneur de cast_db
                    sh '''
                    docker run --name cast_db -d \
                    -e POSTGRES_USER=cast_db_username \
                    -e POSTGRES_PASSWORD=cast_db_password \
                    -e POSTGRES_DB=cast_db_dev \
                    -v postgres_data_cast:/var/lib/postgresql/data/ \
                    postgres:12.1-alpine
                    '''

                    // Attendre que la base de données soit prête
                    sleep(15)  // Ajustez ce délai si nécessaire
                }
            }
        }

        stage('Run Movie Service') {
            steps {
                script {
                    // Démarrer le service movie_service
                    sh '''
                    docker run --name movie-service -d -p 8001:8000 \
                    --link movie_db \
                    -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                    -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ \
                    $DOCKER_ID/movie-service:${DOCKER_TAG} uvicorn app.main:app --host 0.0.0.0 --port 8000
                    '''

                    // Attendre que le service soit prêt
                    sleep(10)  // Ajustez ce délai si nécessaire
                }
            }
        }

        stage('Run Cast Service') {
            steps {
                script {
                    // Démarrer le service cast_service
                    sh '''
                    docker run --name cast-service -d -p 8002:8000 \
                    --link cast_db \
                    -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev \
                    $DOCKER_ID/cast-service:${DOCKER_TAG} uvicorn app.main:app --host 0.0.0.0 --port 8000
                    '''

                    // Attendre que le service soit prêt
                    sleep(10)  // Ajustez ce délai si nécessaire
                }
            }
        }

        stage('Run Acceptance Tests with Curl') {
            steps {
                script {
                    // Test de l'API pour s'assurer que movie_service fonctionne
                    def movieResponse = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8001/api/v1/movies/", returnStdout: true).trim()
                    if (movieResponse != '200') {
                        error "Test d'acceptation échoué pour movie-service: code HTTP ${movieResponse} reçu."
                    }

                    // Test de l'API pour s'assurer que cast_service fonctionne
                    def castResponse = sh(script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:8002/api/v1/casts/", returnStdout: true).trim()
                    if (castResponse != '200') {
                        error "Test d'acceptation échoué pour cast_service: code HTTP ${castResponse} reçu."
                    }
                }
            }
        }

        stage('Stop and Remove Services') {
            steps {
                script {
                    // Arrêter et supprimer les conteneurs
                    sh 'docker stop movie-service cast-service movie_db cast_db && docker rm movie-service cast-service movie_db cast_db'
                }
            }
        }

        stage('Push Docker Images to DockerHub') {
            steps {
                script {
                    // Se connecter à DockerHub
                    sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'

                    // Pousser les images Docker sur DockerHub
                    sh 'docker push $DOCKER_ID/movie-service:${IMAGE_TAG}'
                    sh 'docker push $DOCKER_ID/cast_service:${IMAGE_TAG}'
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }


}
