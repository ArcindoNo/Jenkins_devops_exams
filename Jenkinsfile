pipeline {

    agent any

    // Variables globales du pipeline
    environment {
        DOCKER_HUB_USER = 'datasarcindo'
        CAST_IMAGE      = "${DOCKER_HUB_USER}/cast-service"
        MOVIE_IMAGE     = "${DOCKER_HUB_USER}/movie-service"
    }

    stages {

        // ETAPE 1 : Récupération du code depuis GitHub
        stage('Checkout') {
            steps {
                echo "Récupération du code depuis GitHub..."
                checkout scm
                echo "Code récupéré avec succès !"
            }
        }

        // ETAPE 2 : Construction des images Docker
        stage('Build Docker Images') {
            steps {
                echo "Construction de l image cast-service..."
                sh "docker build -t ${CAST_IMAGE}:${BUILD_NUMBER} -t ${CAST_IMAGE}:latest ./cast-service"

                echo "Construction de l image movie-service..."
                sh "docker build -t ${MOVIE_IMAGE}:${BUILD_NUMBER} -t ${MOVIE_IMAGE}:latest ./movie-service"

                echo "Images construites avec succès !"
            }
        }

        // ETAPE 3 : Envoi des images sur DockerHub
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

        // ETAPE 4 : Déploiement DEV (désactivé pour l instant)
        stage('Deploy to DEV') {
            when {
                expression { return false }
            }
            steps {
                echo "Déploiement en DEV..."
            }
        }

        // ETAPE 5 : Déploiement QA (désactivé pour l instant)
        stage('Deploy to QA') {
            when {
                expression { return false }
            }
            steps {
                echo "Déploiement en QA..."
            }
        }

        // ETAPE 6 : Déploiement STAGING (désactivé pour l instant)
        stage('Deploy to Staging') {
            when {
                expression { return false }
            }
            steps {
                echo "Déploiement en Staging..."
            }
        }

        // ETAPE 7 : Déploiement PROD (désactivé pour l instant)
        // Sera activé uniquement sur un tag Git vX.X depuis master
        stage('Deploy to PROD') {
            when {
                expression { return false }
            }
            steps {
                echo "Déploiement en PROD..."
            }
        }

    }

    // Actions après le pipeline
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
