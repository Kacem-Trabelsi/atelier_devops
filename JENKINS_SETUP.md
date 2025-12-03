# Configuration du Pipeline Jenkins

## Vue d'ensemble

Ce pipeline Jenkins est configuré pour :
1. ✅ Détecter automatiquement les nouveaux commits dans le dépôt Git
2. ✅ Récupérer les dernières mises à jour
3. ✅ Nettoyer et reconstruire le projet (Maven)
4. ✅ Construire l'image Docker
5. ✅ Publier l'image dans un registre Docker

## Méthode 1 : Polling (Déjà configuré)

Le pipeline utilise actuellement le **polling** avec la configuration :
```groovy
triggers {
    pollSCM('* * * * *') // Vérifie toutes les minutes
}
```

**Format Cron** : `minute heure jour mois jour-semaine`
- `* * * * *` = toutes les minutes
- `*/5 * * * *` = toutes les 5 minutes
- `H/15 * * * *` = toutes les 15 minutes (H = hash pour distribuer la charge)

## Méthode 2 : Webhooks avec NGROK (Alternative)

### Étapes pour configurer les webhooks :

1. **Installer NGROK** :
   ```bash
   # Télécharger depuis https://ngrok.com/download
   # Ou avec Chocolatey (Windows)
   choco install ngrok
   ```

2. **Démarrer NGROK** pour exposer Jenkins :
   ```bash
   ngrok http 8080
   # Remplacez 8080 par le port de votre Jenkins
   ```

3. **Copier l'URL HTTPS** fournie par NGROK (ex: `https://abc123.ngrok.io`)

4. **Configurer le webhook dans GitHub** :
   - Allez dans votre dépôt GitHub
   - Settings → Webhooks → Add webhook
   - Payload URL : `https://abc123.ngrok.io/github-webhook/`
   - Content type : `application/json`
   - Events : Sélectionnez "Just the push event"
   - Active : ✅

5. **Modifier le Jenkinsfile** pour utiliser les webhooks :
   ```groovy
   // Supprimer ou commenter la section triggers
   // triggers {
   //     pollSCM('* * * * *')
   // }
   
   // Ajouter dans le stage GIT :
   stage('GIT - Récupération du code') {
       steps {
           script {
               checkout scm
               // ou
               git branch: 'master',
                   url: 'https://github.com/Kacem-Trabelsi/atelier_devops.git'
           }
       }
   }
   ```

6. **Configurer Jenkins pour accepter les webhooks GitHub** :
   - Manage Jenkins → Configure System
   - GitHub section → Ajouter GitHub Server
   - API URL : `https://api.github.com`
   - Credentials : Ajouter votre token GitHub

## Configuration des Credentials dans Jenkins

### 1. Credentials GitHub (jenkins-github-credentials)
- Manage Jenkins → Credentials → Add Credentials
- Type : Username with password ou Secret text (token)
- ID : `jenkins-github-credentials`

### 2. Credentials Docker Registry (docker-registry-credentials)
- Manage Jenkins → Credentials → Add Credentials
- Type : Username with password
- ID : `docker-registry-credentials`
- Username : Votre nom d'utilisateur Docker Hub
- Password : Votre token/password Docker Hub

## Configuration des Outils dans Jenkins

Si vous utilisez la section `tools` dans le Jenkinsfile :

1. **Manage Jenkins → Global Tool Configuration**

2. **JDK** :
   - Name : `JAVA_HOME` ou `JDK17`
   - JAVA_HOME : Chemin vers votre installation JDK 17

3. **Maven** :
   - Name : `M2_HOME`
   - MAVEN_HOME : Chemin vers votre installation Maven

## Variables d'environnement du Pipeline

- `DOCKER_REGISTRY` : URL du registre Docker (par défaut: docker.io pour Docker Hub)
- `DOCKER_IMAGE_NAME` : Nom de l'image (format: username/repository)
- `DOCKER_IMAGE_TAG` : Tag de l'image (utilise BUILD_NUMBER)

## Commandes utiles

### Tester le pipeline localement
```bash
# Build Docker localement
docker build -t student-management:latest .

# Run le conteneur
docker run -p 8089:8089 student-management:latest
```

### Vérifier les logs Jenkins
- Console Output : Affiche tous les logs du build
- Blue Ocean : Interface graphique moderne pour visualiser le pipeline

## Points importants pour l'examen

1. **Déclenchement automatique** : Le pipeline détecte les nouveaux commits
2. **Build Maven** : `mvn clean package` nettoie et reconstruit
3. **Docker** : Build multi-stage pour optimiser l'image
4. **Push Registry** : Publication automatique dans Docker Hub/Registry
5. **Artifacts** : Les JAR sont archivés après le build

## Troubleshooting

- **Erreur "Tool not found"** : Configurez les outils dans Global Tool Configuration
- **Erreur Docker login** : Vérifiez les credentials Docker Registry
- **Webhook ne fonctionne pas** : Vérifiez que NGROK est actif et l'URL est correcte
- **Polling ne détecte pas les changements** : Vérifiez la syntaxe cron et les permissions Git

