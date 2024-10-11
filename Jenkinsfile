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
                    docker stop movie-service cast-service movie_db cast_db castdb moviedb|| true
                    docker rm movie-service cast-service movie_db cast_db castdb moviedb || true
                    '''
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
         stage('cleaning Containers') {
            steps {
                script {
                    sh '''
                    docker stop movie-service cast-service movie_db cast_db castdb moviedb|| true
                    docker rm movie-service cast-service movie_db cast_db castdb moviedb|| true
                    '''
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
                    helm upgrade --install castdb castdb --namespace dev
                    helm upgrade --install moviedb moviedb --namespace dev
                    '''
                    
                    sleep(10)

                    sh '''
                    cp cast-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i 's/jenkins/dev/g' values.yml
                    helm upgrade --install cast-service cast-service --values=values.yml --namespace dev
                    cp movie-service/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    sed -i 's/jenkins/dev/g' values.yml
                    helm upgrade --install movie-service movie-service --values=values.yml --namespace dev

                    '''
                }
            }
        }  
    }
}

