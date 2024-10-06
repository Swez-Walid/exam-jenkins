pipeline {
    environment {
        DOCKER_ID = "swezwalid"
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
    }
    agent any

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_ID/movie-service:${DOCKER_TAG} ./movie-app'
                    sh 'docker build -t $DOCKER_ID/cast-service:${DOCKER_TAG} ./cast-app'
                }
            }
        }

        stage('Stop and Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    docker stop movie-service cast-service movie_db cast_db || true
                    docker rm movie-service cast-service movie_db cast_db || true
                    '''
                }
            }
        }

        stage('Run Movie Database') {
            steps {
                script {
                    sh '''
                    docker run --name movie_db -d \
                    -e POSTGRES_USER=movie_db_username \
                    -e POSTGRES_PASSWORD=movie_db_password \
                    -e POSTGRES_DB=movie_db_dev \
                    -v postgres_data_movie:/var/lib/postgresql/data/ \
                    postgres:12.1-alpine
                    '''
                    sleep(15)
                }
            }
        }

        stage('Run Cast Database') {
            steps {
                script {
                    sh '''
                    docker run --name cast_db -d \
                    -e POSTGRES_USER=cast_db_username \
                    -e POSTGRES_PASSWORD=cast_db_password \
                    -e POSTGRES_DB=cast_db_dev \
                    -v postgres_data_cast:/var/lib/postgresql/data/ \
                    postgres:12.1-alpine
                    '''
                    sleep(15)
                }
            }
        }

        stage('Run Movie Service') {
            steps {
                script {
                    sh '''
                    docker run --name movie-service -d -p 8001:8000 \
                    --link movie_db \
                    -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev \
                    -e CAST_SERVICE_HOST_URL=http://cast-service:8000/api/v1/casts/ \
                    $DOCKER_ID/movie-service:${DOCKER_TAG} uvicorn app.main:app --host 0.0.0.0 --port 8000
                    '''
                    sleep(10)
                }
            }
        }

        stage('Run Cast Service') {
            steps {
                script {
                    sh '''
                    docker run --name cast-service -d -p 8002:8000 \
                    --link cast_db \
                    -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev \
                    $DOCKER_ID/cast-service:${DOCKER_TAG} uvicorn app.main:app --host 0.0.0.0 --port 8000
                    '''
                    sleep(10)
                }
            }
        }

        

        stage('Push Docker Images to DockerHub') {
            steps {
                script {
                    sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
                    sh 'docker push $DOCKER_ID/movie-service:${DOCKER_TAG}'
                    sh 'docker push $DOCKER_ID/cast-service:${DOCKER_TAG}'
                }
            }
        }

        stage('Deploiement en dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp cast-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install cast-service cast-service --values=values.yml --namespace dev
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install movie-service movie-service --values=values.yml --namespace dev
                    helm upgrade --install castdb castdb --namespace dev
                    helm upgrade --install moviedb moviedb --namespace dev
                    '''
                }
            }
        }

        stage('Deploiement en QA') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp cast-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install cast-service cast-service --values=values.yml --namespace QA
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install movie-service movie-service --values=values.yml --namespace QA
                    helm upgrade --install castdb castdb --namespace QA
                    helm upgrade --install moviedb moviedb --namespace QA
                    '''
                }
            }
        }

        stage('Deploiement en prod') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    cat $KUBECONFIG > .kube/config
                    cp cast-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install cast-service cast-service --values=values.yml --namespace prod
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install movie-service movie-service --values=values.yml --namespace prod
                    helm upgrade --install castdb castdb --namespace prod
                    helm upgrade --install moviedb moviedb --namespace prod
                    '''
                }
            }
        }
    }
}

