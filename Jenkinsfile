pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/rishabhsharma5554/Shopping-Cart.git'
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Shopping-Cart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=Shopping-Cart '''
               }
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '83dacb31-bac7-497e-b3d1-945a99a3fa72', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart rishabhsh5554/shopping-cart:latest"
                        sh "docker push rishabhsh5554/shopping-cart:latest"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '83dacb31-bac7-497e-b3d1-945a99a3fa72', toolName: 'docker') {
                        sh "docker run -d --name shopping-app -p 8500:8500 rishabhsh5554/shopping-cart"
                    }
                }
            }
        }
    }
}
