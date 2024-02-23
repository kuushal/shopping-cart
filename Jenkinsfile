pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk8'

    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kuushal/shopping-cart.git'
            }
        }
        stage('Compile') {
            steps {
                bat "mvn compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                bat 'mvn test'
            }
        }
        
        stage('SonarQube Analysis') {
            tools {
                jdk 'jdk11'            
            }
            steps {
                withSonarQubeEnv('sonar') {
                     bat 'mvn sonar:sonar'
                //   bat " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Shopping-Cart -Dsonar.projectName=Shopping-Cart -Dsonar.java.binaries=. "
                }
            }
        }
        
        stage('OWASP') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

          stage('Build') {
            steps {
                bat 'mvn package'
            }
        }

          stage('Deploy the artifact to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk8', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    bat "mvn deploy"
                }
            }
        }

          stage('Build & Tag Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-credentials') {
                        bat 'docker build -t virigo3782/shopping-cart:latest -f docker/Dockerfile .'
                        
                    }
               }
            }
        }

         stage('Trivy scan') {
            steps {
                bat 'trivy image virigo3782/shopping-cart:latest > trivy-report.txt'
            }
        }  
    
         stage('Push Docker Image to docker registry') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-credentials') {
                        bat 'docker push virigo3782/shopping-cart'
                        
                    }
               }
            }
        }
        
        stage('Deploy to kubernetes a') {
            steps {
                script{
                     bat 'kubectl apply -f deploymentservice.yml'
                }
            }
        }  
        
    }
}
