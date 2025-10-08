pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'petclinic-app'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Récupération du code source...'
                checkout scm
            }
        }
        
        stage('Build with Maven') {
            steps {
                echo '🔨 Compilation du projet...'
                sh '''
                    chmod +x ./mvnw
                    ./mvnw clean package -DskipTests
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo '🧪 Exécution des tests...'
                sh '''
                    ./mvnw test
                '''
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐳 Construction de l\'image Docker...'
                sh '''
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l\'application...'
                sh '''
                    # Arrêter le conteneur existant s'il existe
                    docker stop petclinic || true
                    docker rm petclinic || true
                    
                    # Démarrer le nouveau conteneur
                    docker run -d \
                        --name petclinic \
                        --network test-compose_app-network \
                        -p 8080:8080 \
                        ${DOCKER_IMAGE}:${DOCKER_TAG}
                '''
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline exécuté avec succès!'
            echo '🌐 Application accessible sur http://localhost:8080'
        }
        failure {
            echo '❌ Le pipeline a échoué'
        }
        always {
            echo '🧹 Nettoyage...'
            sh 'docker system prune -f || true'
        }
    }
}
