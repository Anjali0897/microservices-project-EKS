pipeline {
    agent any
        tools{
            jdk 'jdk17'
        } 
    stages{
        stage('Build') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew clean build'
            }
        }
        stage('CQA') {
            steps {
                script {
                    def scannerHome = tool 'mysonar'

                    withSonarQubeEnv('mysonar') {
                        sh "mvn clean verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=myproject"
                    }
                }
            }
        }

        stage('QualityGate') {
            steps {
                waitForQualityGate abortPipeline: true,
                    credentialsId: 'sonar-password'
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker-cred',
                        toolName: 'docker'
                    ) {
                        sh 'docker build -t anjaligundavaram/adservice:latest .'
                    }
                }
            }
        }

        stage('Trivy') {
            steps {
                sh 'trivy image anjaligundavaram/adservice:latest >> adservice-reports.txt'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(
                        credentialsId: 'docker-cred',
                        toolName: 'docker'
                    ) {
                        sh 'docker push anjaligundavaram/adservice:latest'
                    }
                }
            }
        }
    }
}
