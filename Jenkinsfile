pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'petclinic-app'
        CONTAINER_NAME = 'petclinic-container'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '🔄 Récupération du code source...'
                checkout scm
            }
        }
        
        stage('Build with Maven in Docker') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                    args '-v /var/jenkins_home/.m2:/root/.m2'
                }
            }
            steps {
                echo '🔨 Compilation du projet avec Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐳 Construction de l\'image Docker...'
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Stop Old Container') {
            steps {
                echo '🛑 Nettoyage de tous les conteneurs petclinic...'
                script {
                    sh '''
                        # Supprimer TOUS les conteneurs avec le nom (même ceux en état Created)
                        docker ps -aq --filter "name=petclinic-container" | xargs -r docker rm -f || true
                        
                        # Vérifier que le port est libre
                        echo "✅ Nettoyage terminé"
                        docker ps | grep 8081 || echo "✅ Port 8081 disponible"
                    '''
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo '🚀 Déploiement de l\'application...'
                script {
                    sh """
                        docker run -d \
                            --name ${CONTAINER_NAME} \
                            --network test-compose_app-network \
                            -p 8081:8080 \
                            ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Health Check') {
            steps {
                echo '🏥 Vérification de la santé de l\'application...'
                script {
                    sh '''
                        echo "Attente du démarrage de l'application..."
                        sleep 30
                        
                        echo "Test de connexion..."
                        curl -f http://localhost:8081 || exit 1
                        
                        echo "✅ Application démarrée avec succès !"
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline terminé avec succès !'
            echo '🌐 Application disponible sur http://localhost:8081'
        }
        failure {
            echo '❌ Le pipeline a échoué'
        }
        always {
            echo '🧹 Nettoyage des images inutilisées...'
            sh 'docker image prune -f || true'
        }
    }
}
