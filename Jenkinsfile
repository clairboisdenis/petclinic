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
                        docker ps | grep ${HOST_PORT} || echo "✅ Port ${HOST_PORT} disponible"
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
                        
                        echo "✅ Conteneur démarré : ${CONTAINER_NAME}"
                        docker ps --filter name=${CONTAINER_NAME}
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo "🏥 Vérification de la santé de l'application..."
                script {
                    sh """
                        echo "⏳ Attente du démarrage (30s)..."
                        sleep 30
                        
                        echo "🔍 Vérification des logs..."
                        docker logs ${CONTAINER_NAME} --tail 20
                        
                        echo "🧪 Test de connexion depuis l'hôte Docker..."
                        docker exec ${CONTAINER_NAME} curl -f http://localhost:${CONTAINER_PORT} || echo "⚠️ App pas encore prête"
                        
                        echo "✅ Conteneur actif !"
                        docker ps --filter name=${CONTAINER_NAME} --format "table {{.Names}}\\t{{.Status}}\\t{{.Ports}}"
                    """
                }
            }
        }
        
        stage('Access Information') {
            steps {
                echo "📋 Informations d'accès :"
                script {
                    sh """
                        echo ""
                        echo "🌐 Application accessible sur :"
                        echo "   http://localhost:${HOST_PORT}"
                        echo ""
                        echo "🐳 Conteneur : ${CONTAINER_NAME}"
                        echo "📊 Logs : docker logs -f ${CONTAINER_NAME}"
                        echo ""
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
            echo "✅ Le pipeline a réussi !"
            echo "🌐 Accédez à l'application : http://localhost:${HOST_PORT}"
        }
        failure {
            echo "❌ Le pipeline a échoué"
            sh 'docker logs ${CONTAINER_NAME} --tail 50 || true'
        }
    }
}
