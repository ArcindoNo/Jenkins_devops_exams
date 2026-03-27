pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'datasarcindo'
        CAST_IMAGE      = "${DOCKER_HUB_USER}/cast-service"
        MOVIE_IMAGE     = "${DOCKER_HUB_USER}/movie-service"
        HELM_CHART      = './charts'
    }

    stages {

        // ─────────────────────────────────────────
        // STAGE 1 : Récupération du code
        // ─────────────────────────────────────────
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ─────────────────────────────────────────
        // STAGE 2 : Build des images Docker
        // ─────────────────────────────────────────
        stage('Build Docker Images') {
            steps {
                script {
                    sh """
                        docker build -t ${CAST_IMAGE}:${BUILD_NUMBER} -t ${CAST_IMAGE}:latest ./cast-service
                        docker build -t ${MOVIE_IMAGE}:${BUILD_NUMBER} -t ${MOVIE_IMAGE}:latest ./movie-service
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 3 : Push sur DockerHub
        // ─────────────────────────────────────────
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${CAST_IMAGE}:${BUILD_NUMBER}
                        docker push ${CAST_IMAGE}:latest
                        docker push ${MOVIE_IMAGE}:${BUILD_NUMBER}
                        docker push ${MOVIE_IMAGE}:latest
                        docker logout
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 4 : Déploiement en DEV
        // ─────────────────────────────────────────
        stage('Deploy to DEV') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh """
                        helm upgrade --install cast-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace dev \
                            --set image.repository=${CAST_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30001

                        helm upgrade --install movie-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace dev \
                            --set image.repository=${MOVIE_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30002
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 5 : Déploiement en QA
        // ─────────────────────────────────────────
        stage('Deploy to QA') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh """
                        helm upgrade --install cast-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace qa \
                            --set image.repository=${CAST_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30003

                        helm upgrade --install movie-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace qa \
                            --set image.repository=${MOVIE_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30004
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 6 : Déploiement en STAGING
        // ─────────────────────────────────────────
        stage('Deploy to Staging') {
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh """
                        helm upgrade --install cast-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace staging \
                            --set image.repository=${CAST_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30005

                        helm upgrade --install movie-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace staging \
                            --set image.repository=${MOVIE_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30006
                    """
                }
            }
        }

        // ─────────────────────────────────────────
        // STAGE 7 : Déploiement en PROD
        // Uniquement sur un tag Git (v*) depuis master
        // ─────────────────────────────────────────
        stage('Deploy to PROD') {
            when {
                allOf {
                    branch 'master'
                    tag pattern: 'v.*', comparator: 'REGEXP'
                }
            }
            steps {
                withCredentials([file(
                    credentialsId: 'kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh """
                        helm upgrade --install cast-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace prod \
                            --set image.repository=${CAST_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30007

                        helm upgrade --install movie-service ${HELM_CHART} \
                            --kubeconfig=$KUBECONFIG \
                            --namespace prod \
                            --set image.repository=${MOVIE_IMAGE} \
                            --set image.tag=${BUILD_NUMBER} \
                            --set service.nodePort=30008
                    """
                }
            }
        }
    }

    // ─────────────────────────────────────────
    // POST : Notifications de fin de pipeline
    // ─────────────────────────────────────────
    post {
        success {
            echo " Pipeline terminé avec succès ! Build #${BUILD_NUMBER}"
        }
        failure {
            echo " Pipeline en échec ! Vérifiez les logs du build #${BUILD_NUMBER}"
        }
        always {
            sh 'docker logout || true'
        }
    }
}
