pipeline {
    agent any
    
    tools{
        jdk "jdk17"
        maven "maven3"
    }
    
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }
    
    

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'Git-Cred', url: 'https://github.com/amitsahu0/BoardgameListingWebApp.git'
                }
            }
        }
        
        stage('Compile Code') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy File scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        
        stage('Build') {
            steps {
               sh "mvn package"
            }
        }
        
        stage('Publish To Nexus') {
            steps {
               withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
        
        stage('Build & Tag and Push Docker Image') {
            steps {
               script {
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t amitdocker6/boardgame:latest ."
                            sh "docker push amitdocker6/boardgame:latest "
                    }
                }
            }
        }
        
        stage('Deploy to K8s') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.46.128:6443') {
                    sh " kubectl apply -f deployment-service.yml"    
                }
            }
        }
    
        stage('check K8s pods') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.46.128:6443') {
                        sh " kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                }
            }
        }

}

