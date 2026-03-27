pipeline {

    agent any

    // Variables globales du pipeline
    environment {
        DOCKER_HUB_USER = 'datasarcindo'
        CAST_IMAGE      = "${DOCKER_HUB_USER}/cast-service"
        MOVIE_IMAGE     = "${DOCKER_HUB_USER}/movie-service"
        HELM_CHART      = "${WORKSPACE}/charts"
    }

    stages {

        // ─────────────────────────────────────────
        // ETAPE 1 : Récupération du code depuis GitHub
        // ─────────────────────────────────────────
        stage('Checkout') {
            steps {
                echo "Récupération du code depuis GitHub..."
                checkout scm
                echo "Code récupéré avec succès !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 2 : Construction des images Docker
        // ─────────────────────────────────────────
        stage('Build Docker Images') {
            steps {
                echo "Construction de l image cast-service..."
                sh "docker build -t ${CAST_IMAGE}:${BUILD_NUMBER} -t ${CAST_IMAGE}:latest ./cast-service"

                echo "Construction de l image movie-service..."
                sh "docker build -t ${MOVIE_IMAGE}:${BUILD_NUMBER} -t ${MOVIE_IMAGE}:latest ./movie-service"

                echo "Images construites avec succès !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 3 : Envoi des images sur DockerHub
        // ─────────────────────────────────────────
        stage('Push to DockerHub') {
            steps {
                echo "Connexion à DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: 'DOCKER_HUB_PASS',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"

                    echo "Envoi de l image cast-service..."
                    sh "docker push ${CAST_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${CAST_IMAGE}:latest"

                    echo "Envoi de l image movie-service..."
                    sh "docker push ${MOVIE_IMAGE}:${BUILD_NUMBER}"
                    sh "docker push ${MOVIE_IMAGE}:latest"

                    sh "docker logout"
                    echo "Images envoyées sur DockerHub avec succès !"
                }
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 4 : Déploiement en DEV
        // Déclenché sur chaque push sur master
        // ─────────────────────────────────────────
        stage('Deploy to DEV') {
            steps {
                echo "Déploiement en DEV..."
                sh """
                    helm upgrade --install cast-service ${HELM_CHART} \
                        --namespace dev \
                        --set image.repository=${CAST_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30001
                """
                sh """
                    helm upgrade --install movie-service ${HELM_CHART} \
                        --namespace dev \
                        --set image.repository=${MOVIE_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30002
                """
                echo "Déploiement en DEV terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 5 : Déploiement en QA
        // Déclenché sur chaque push sur master
        // ─────────────────────────────────────────
        stage('Deploy to QA') {
            steps {
                echo "Déploiement en QA..."
                sh """
                    helm upgrade --install cast-service ${HELM_CHART} \
                        --namespace qa \
                        --set image.repository=${CAST_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30003
                """
                sh """
                    helm upgrade --install movie-service ${HELM_CHART} \
                        --namespace qa \
                        --set image.repository=${MOVIE_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30004
                """
                echo "Déploiement en QA terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 6 : Déploiement en STAGING
        // Déclenché sur chaque push sur master
        // ─────────────────────────────────────────
        stage('Deploy to Staging') {
            steps {
                echo "Déploiement en Staging..."
                sh """
                    helm upgrade --install cast-service ${HELM_CHART} \
                        --namespace staging \
                        --set image.repository=${CAST_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30005
                """
                sh """
                    helm upgrade --install movie-service ${HELM_CHART} \
                        --namespace staging \
                        --set image.repository=${MOVIE_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30006
                """
                echo "Déploiement en Staging terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 7 : Déploiement en PROD
        // Déclenché UNIQUEMENT sur un tag Git vX.X
        // depuis la branche master
        // ─────────────────────────────────────────
        stage('Deploy to PROD') {
            when {
                allOf {
                    branch 'master'
                    tag pattern: 'v.*', comparator: 'REGEXP'
                }
            }
            steps {
                echo "Déploiement en PROD..."
                sh """
                    helm upgrade --install cast-service ${HELM_CHART} \
                        --namespace prod \
                        --set image.repository=${CAST_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30007
                """
                sh """
                    helm upgrade --install movie-service ${HELM_CHART} \
                        --namespace prod \
                        --set image.repository=${MOVIE_IMAGE} \
                        --set image.tag=${BUILD_NUMBER} \
                        --set service.nodePort=30008
                """
                echo "Déploiement en PROD terminé !"
            }
        }

    }

    // ─────────────────────────────────────────
    // Actions après le pipeline
    // ─────────────────────────────────────────
    post {
        success {
            echo "Pipeline terminé avec succès ! Build numéro ${BUILD_NUMBER}"
        }
        failure {
            echo "Pipeline en échec ! Consultez les logs du build ${BUILD_NUMBER}"
        }
        always {
            sh "docker logout || true"
        }
    }

}