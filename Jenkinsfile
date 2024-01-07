pipeline {
    agent any
    
    tools{
        jdk 'jdk17' 
    }
    
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Github Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/viddhant1205/Python-Webapp.git'
            }
        }
        
        stage('OWASP ') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Python-Webapp \
                      -Dsonar.projectKey=Pyhton-Webapp '''
               }
            }
        }
        
        stage('Docker build & Tag') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "make image"
                   }
               }
           } 
        }
        
        stage('Trrivy Image Scan') {
            steps {
                sh "trivy image kubegourav/python-webapp:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "make push"
                   }
               }
           } 
        }
        
        stage('Deploy to Docker Container') {
            steps {
               script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                     sh "docker run -d -p 5000:5000 kubegourav/python-webapp:latest"
                   }
               }
           } 
        }
        
    }
 }
