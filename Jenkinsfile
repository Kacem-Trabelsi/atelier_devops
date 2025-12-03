pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    
    // Déclenchement automatique : vérification toutes les minutes pour détecter les nouveaux commits
    triggers {
        pollSCM('* * * * *') // Polling toutes les minutes (format cron: minute heure jour mois jour-semaine)
    }
    
    environment {
        // Configuration Docker Registry (à adapter selon votre registre)
        DOCKER_REGISTRY = 'docker.io' // ou 'registry.example.com' pour un registre privé
        DOCKER_IMAGE_NAME = 'kacem-trabelsi/student-management'
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('GIT - Récupération du code') {
            steps {
                script {
                    echo 'Récupération des dernières mises à jour du dépôt Git...'
                    git branch: 'master',
                        url: 'https://github.com/Kacem-Trabelsi/atelier_devops.git',
                        credentialsId: 'jenkins-github-credentials'
                    sh 'git log -1 --oneline'
                }
            }
        }
        
        stage('Build - Nettoyage et compilation') {
            steps {
                script {
                    echo 'Nettoyage et reconstruction du projet...'
                    sh 'mvn clean compile'
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo 'Exécution des tests avec profil test (H2 en mémoire)...'
                    sh 'mvn test -Dspring.profiles.active=test'
                }
            }
            post {
                always {
                    // Publier les résultats des tests
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package - Création du JAR') {
            steps {
                script {
                    echo 'Création du package JAR...'
                    sh 'mvn package -DskipTests'
                }
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
        
        stage('Docker - Build de l\'image') {
            steps {
                script {
                    echo "Construction de l'image Docker..."
                    sh """
                        docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                        docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest
                    """
                }
            }
        }
        
        stage('Docker - Push vers le registre') {
            steps {
                script {
                    echo "Publication de l'image Docker dans le registre..."
                    withCredentials([usernamePassword(credentialsId: 'docker-registry-credentials', 
                                                      usernameVariable: 'DOCKER_USER', 
                                                      passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo \$DOCKER_PASS | docker login ${DOCKER_REGISTRY} -u \$DOCKER_USER --password-stdin
                            docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                            docker push ${DOCKER_IMAGE_NAME}:latest
                        """
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline exécuté avec succès!'
            echo "Image Docker disponible: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline échoué. Vérifiez les logs pour plus de détails.'
        }
        always {
            echo 'Nettoyage des ressources...'
            // Optionnel: nettoyer les images Docker locales
            // sh 'docker image prune -f'
        }
    }
}
