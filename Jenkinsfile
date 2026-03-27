/*
 * Pipeline DevOps pour le projet de formation
 *
 * Étapes :
 * 1. Checkout : Récupération du code depuis GitHub
 * 2. Build Docker Images : Construction des images Docker pour cast-service et movie-service
 * 3. Test Docker Images : Test des images Docker pour vérifier leur bon fonctionnement
 * 4. Push to DockerHub : Envoi des images sur DockerHub
 * 5. Deploy to DEV/QA/STAGING : Déploiement dans les environnements de test
 * 6. Deploy to PROD : Déploiement en production (manuel, sur tag v.*)
 *
 * Variables d'environnement :
 * - DOCKER_HUB_USER : Utilisateur DockerHub
 * - CAST_IMAGE : Nom de l'image Docker pour cast-service
 * - MOVIE_IMAGE : Nom de l'image Docker pour movie-service
 * - HELM_CHART : Chemin vers les charts Helm
 */

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
                echo "Construction de l'image cast-service..."
                sh "docker build -t ${CAST_IMAGE}:${BUILD_NUMBER} -t ${CAST_IMAGE}:latest ./cast-service"

                echo "Construction de l'image movie-service..."
                sh "docker build -t ${MOVIE_IMAGE}:${BUILD_NUMBER} -t ${MOVIE_IMAGE}:latest ./movie-service"

                echo "Images construites avec succès !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 2.5 : Test des images Docker
        // ─────────────────────────────────────────
        stage('Test Docker Images') {
            steps {
                echo "Test de l'image cast-service..."
                sh "docker run -d --name test-cast ${CAST_IMAGE}:${BUILD_NUMBER}"
                sh "docker logs test-cast"
                sh "docker rm -f test-cast"

                echo "Test de l'image movie-service..."
                sh "docker run -d --name test-movie ${MOVIE_IMAGE}:${BUILD_NUMBER}"
                sh "docker logs test-movie"
                sh "docker rm -f test-movie"

                echo "Tests des images terminés avec succès !"
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
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                }
                echo "Envoi des images..."
                sh "docker push ${CAST_IMAGE}:${BUILD_NUMBER}"
                sh "docker push ${CAST_IMAGE}:latest"
                sh "docker push ${MOVIE_IMAGE}:${BUILD_NUMBER}"
                sh "docker push ${MOVIE_IMAGE}:latest"
                sh "docker logout"
            }
        }

        // Fonction pour déployer dans un environnement donné
        def deployToEnv(String namespace, int castPort, int moviePort) {
            sh """
                helm upgrade --install cast-service ${HELM_CHART} \
                    --namespace ${namespace} \
                    --set image.repository=${CAST_IMAGE} \
                    --set image.tag=${BUILD_NUMBER} \
                    --set service.nodePort=${castPort}
            """
            sh """
                helm upgrade --install movie-service ${HELM_CHART} \
                    --namespace ${namespace} \
                    --set image.repository=${MOVIE_IMAGE} \
                    --set image.tag=${BUILD_NUMBER} \
                    --set service.nodePort=${moviePort}
            """
        }

        // ─────────────────────────────────────────
        // ETAPE 4 : Déploiement en DEV
        // ─────────────────────────────────────────
        stage('Deploy to DEV') {
            steps {
                echo "Déploiement en DEV..."
                deployToEnv('dev', 30001, 30002)
                echo "Déploiement en DEV terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 5 : Déploiement en QA
        // ─────────────────────────────────────────
        stage('Deploy to QA') {
            steps {
                echo "Déploiement en QA..."
                deployToEnv('qa', 30003, 30004)
                echo "Déploiement en QA terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 6 : Déploiement en STAGING
        // ─────────────────────────────────────────
        stage('Deploy to Staging') {
            steps {
                echo "Déploiement en Staging..."
                deployToEnv('staging', 30005, 30006)
                echo "Déploiement en Staging terminé !"
            }
        }

        // ─────────────────────────────────────────
        // ETAPE 7 : Déploiement en PROD
        // Déclenché UNIQUEMENT sur un tag Git vX.X
        // ─────────────────────────────────────────
        stage('Deploy to PROD') {
            when {
                tag pattern: 'v.*', comparator: 'REGEXP'
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