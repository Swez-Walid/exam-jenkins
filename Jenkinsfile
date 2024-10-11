
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

        stage('Push Docker Images to DockerHub') {
            steps {
                script {
                    sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
                    sh 'docker push $DOCKER_ID/movie-service:${DOCKER_TAG}'
                    sh 'docker push $DOCKER_ID/cast-service:${DOCKER_TAG}'
                }
            }
        }

        stage('Deploy to dev') {
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    helm upgrade --install movie-db ./moviedb --namespace dev --set namespace=dev
                    helm upgrade --install cast-db ./castdb --namespace dev --set namespace=dev
                    helm upgrade --install movie-service ./movie-service --namespace dev --set image.tag=${DOCKER_TAG} --set namespace=dev
                    helm upgrade --install cast-service ./cast-service --namespace dev --set image.tag=${DOCKER_TAG} --set namespace=dev
                    '''
                }
            }
        }

        stage('Deploy to qa') {
            when {
                branch 'qa'
            }
            steps {
                script {
                    sh '''
                    helm upgrade --install movie-db ./moviedb --namespace qa --set namespace=qa
                    helm upgrade --install cast-db ./castdb --namespace qa --set namespace=qa
                    helm upgrade --install movie-service ./movie-service --namespace qa --set image.tag=${DOCKER_TAG} --set namespace=qa
                    helm upgrade --install cast-service ./cast-service --namespace qa --set image.tag=${DOCKER_TAG} --set namespace=qa
                    helm upgrade --install nginx-ingress ./nginx --namespace qa
                    '''
                }
            }
        }

        stage('Deploy to staging') {
            when {
                branch 'staging'
            }
            steps {
                script {
                    sh '''
                    helm upgrade --install movie-db ./moviedb --namespace staging --set namespace=staging
                    helm upgrade --install cast-db ./castdb --namespace staging --set namespace=staging
                    helm upgrade --install movie-service ./movie-service --namespace staging --set image.tag=${DOCKER_TAG} --set namespace=staging
                    helm upgrade --install cast-service ./cast-service --namespace staging --set image.tag=${DOCKER_TAG} --set namespace=staging
                    helm upgrade --install nginx-ingress ./nginx --namespace staging
                    '''
                }
            }
        }

        stage('Deploy to prod') {
            when {
                branch 'master'
            }
            steps {
                script {
                    sh '''
                    helm upgrade --install movie-db ./moviedb --namespace prod --set namespace=prod
                    helm upgrade --install cast-db ./castdb --namespace prod --set namespace=prod
                    helm upgrade --install movie-service ./movie-service --namespace prod --set image.tag=${DOCKER_TAG} --set namespace=prod
                    helm upgrade --install cast-service ./cast-service --namespace prod --set image.tag=${DOCKER_TAG} --set namespace=prod
                    helm upgrade --install nginx-ingress ./nginx --namespace prod
                    '''
                }
            }
        }
    }
}
