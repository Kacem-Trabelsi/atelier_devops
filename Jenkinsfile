pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JAVA_HOME'
    }
    
    stages {
        stage('GIT') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/Kacem-Trabelsi/atelier_devops.git',
                    credentialsId: 'jenkins-github-credentials'
            }
        }
    }
}

