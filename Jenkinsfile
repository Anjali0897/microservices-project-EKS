pipeline {
    agent {
        label 'slave-master'
    }

    tools {
        maven 'mymaven'
    }

    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
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
                        sh "docker build -t anjaligundavaram/currencyservice:latest ."
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push anjaligundavaram/currencyservice:latest "
                    }
                }
            }
        }
    }
}
