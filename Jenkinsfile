pipeline {
    agent any

    environment {
        DOCKER_ID      = "datasarcindo"
        CAST_IMAGE     = "cast-service"
        MOVIE_IMAGE    = "movie-service"
        DOCKER_TAG     = "v.${BUILD_ID}.0"
        KUBECONFIG     = credentials('kubeconfig')
    }

    stages {

        stage('Build Cast Service') {
            steps {
                sh """
                    docker build -t ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG} ./cast-service
                """
            }
        }

        stage('Build Movie Service') {
            steps {
                sh """
                    docker build -t ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG} ./movie-service
                """
            }
        }

        stage('Push Cast Service') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_HUB_PASS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }

        stage('Push Movie Service') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_HUB_PASS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        docker push ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                sh """
                    helm upgrade --install cast-service ./charts \
                        --namespace dev \
                        --set image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}

                    helm upgrade --install movie-service ./charts \
                        --namespace dev \
                        --set image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}
                """
            }
        }

        stage('Deploy to QA') {
            steps {
                sh """
                    helm upgrade --install cast-service ./charts \
                        --namespace qa \
                        --set image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}

                    helm upgrade --install movie-service ./charts \
                        --namespace qa \
                        --set image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}
                """
            }
        }

        stage('Deploy to Staging') {
            steps {
                sh """
                    helm upgrade --install cast-service ./charts \
                        --namespace staging \
                        --set image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}

                    helm upgrade --install movie-service ./charts \
                        --namespace staging \
                        --set image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}
                """
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Déployer en production ?', ok: 'Oui, déployer !'
                }
                sh """
                    helm upgrade --install cast-service ./charts \
                        --namespace prod \
                        --set image.repository=${DOCKER_ID}/${CAST_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}

                    helm upgrade --install movie-service ./charts \
                        --namespace prod \
                        --set image.repository=${DOCKER_ID}/${MOVIE_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --kubeconfig ${KUBECONFIG}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline terminé avec succès !'
        }
        failure {
            echo 'Pipeline en échec !'
        }
    }
}