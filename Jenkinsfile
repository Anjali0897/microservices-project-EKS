
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

        stage('CQA') {
            steps {
                script {
                    def scannerHome = tool 'mysonar'

                    withSonarQubeEnv('mysonar') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=myproject -Dsonar.sources=. -Dsonar.java.binaries=target/classes"
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
