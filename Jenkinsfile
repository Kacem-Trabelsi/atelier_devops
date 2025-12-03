pipeline {
    agent any
    
    tools {
        maven 'M2_HOME'
        jdk 'JDK17'
    }
    
    stages {
        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Kacem-Trabelsi/devops.git',
                    credentialsId: 'jenkins-github-credentials'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            cleanWs()
        }
    }
}

