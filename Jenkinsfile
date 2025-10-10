pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = "petclinic-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        CONTAINER_NAME = "petclinic-container"
        HOST_PORT = "8081"
        CONTAINER_PORT = "8080"
        NETWORK = "test-compose_app-network"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo "🔄 Récupération du code source..."
                checkout scm
            }
        }
        
        stage('Build with Maven in Docker') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                echo "🔨 Compilation du projet avec Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo "🐳 Construction de l'image Docker..."
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Stop Old Container') {
            steps {
                echo "🛑 Nettoyage de tous les conteneurs petclinic..."
                script {
                    sh """
                        docker ps -aq --filter name=${CONTAINER_NAME} | xargs -r docker rm -f || true
                        echo "✅ Nettoyage terminé"
                    """
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo "🚀 Déploiement de l'application..."
                script {
                    sh """
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            --network ${NETWORK} \
                            -p ${HOST_PORT}:${CONTAINER_PORT} \
                            ${DOCKER_IMAGE}:latest
                        
                        echo "✅ Conteneur démarré"
                        docker ps --filter name=${CONTAINER_NAME}
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo "🏥 Vérification de la santé..."
                script {
                    sh """
                        echo "⏳ Attente 30s pour démarrage..."
                        sleep 30
                        
                        echo ""
                        echo "🔍 Logs du conteneur :"
                        docker logs ${CONTAINER_NAME} --tail 20
                        
                        echo ""
                        echo "🧪 Test HTTP interne :"
                        docker exec ${CONTAINER_NAME} curl -f http://localhost:${CONTAINER_PORT} > /dev/null 2>&1 && echo "✅ Application répond !" || echo "⚠️ Application pas encore prête"
                        
                        echo ""
                        echo "📊 Statut conteneur :"
                        docker ps --filter name=${CONTAINER_NAME} --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                    """
                }
            }
        }
        
        stage('Access Info') {
            steps {
                echo "📋 Informations d'accès"
                script {
                    sh """
                        echo ""
                        echo "========================================="
                        echo "🌐 Application accessible sur :"
                        echo "   http://localhost:${HOST_PORT}"
                        echo ""
                        echo "🐳 Conteneur : ${CONTAINER_NAME}"
                        echo "📊 Voir logs : docker logs -f ${CONTAINER_NAME}"
                        echo "========================================="
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo "🧹 Nettoyage des images inutilisées..."
            sh 'docker image prune -f'
        }
        success {
            echo "✅ LE PIPELINE A RÉUSSI !"
            echo "🌐 Accédez à l'application : http://localhost:${HOST_PORT}"
        }
        failure {
            echo "❌ LE PIPELINE A ÉCHOUÉ"
            echo "📊 Logs du conteneur :"
            sh 'docker logs ${CONTAINER_NAME} --tail 50 || echo "Conteneur introuvable"'
        }
    }
}
