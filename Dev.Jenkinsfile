pipeline {
    agent any
    
    tools {
        nodejs 'node21'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
               git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/iamsaurav-karki/3-Tier-Full-Stack.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Unit Tests') {
            steps {
                sh "npm test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html ."
            }
        }
        stage('SonarQube analysis') {
            steps {
                 withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                     -Dsonar.projectKey=Campground \
                     -Dsonar.projectName=Campground \
                     -Dsonar.sources=.
                    '''
            }
          }
        }

        stage('Docker Build and Tag') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker build -t sauravkarki/camp:latest ."
             }
                }
                
            }
        }
        
        stage('Trivy Docker image Scan') {
            steps {
                sh "trivy image --format table -o fs-report.html sauravkarki/camp:latest"
            }
        }
        
        stage('Docker push image') {
            steps {
                script{
                     withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker push sauravkarki/camp:latest"
                }
            }
            }
        }
        
        stage('Docker Deploy To Dev') {
            steps {
                script{
                     withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    sh "docker run -d -p 3000:3000 sauravkarki/camp:latest"
                }
            }
            }
        }
    }
}
