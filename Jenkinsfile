pipeline {
    agent any

    stages {
        stage('CQA'){
            steps{
                script{
                    def scannerHome = tool 'mysonar'
                    withSonarQubeEnv('mysonar'){
                         sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=."
                    }
                }
            }
        }
        stage('QualityGate'){
            steps{
                waitForQualityGate abortPipeline: true, credentialsId: 'sonar-password'
            }
        }  
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t anjaligundavaram/adservice:latest ."
                    }
                }
            }
        }
        stage('Trivy'){
            steps{
                sh 'trivy image adservice >> adservice-reports.txt'
            }
        }  
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push anjaligundavaram/adservice:latest "
                    }
                }
            }
        }
    }
}
