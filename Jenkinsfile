pipeline {
    agent any
    stages{
        stage('CQA') {
            steps {
                script {
                    withSonarQubeEnv('mysonar') {
                        dir('src') {
                            sh '''
                               dotnet sonarscanner begin /k:"myproject" \
                               /d:sonar.host.url="$SONAR_HOST_URL" \
                               /d:sonar.token="$SONAR_AUTH_TOKEN"
                               dotnet build cartservice.csproj
                               dotnet sonarscanner end /d:sonar.token="$SONAR_AUTH_TOKEN"
                              '''
                           }
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
                    dir('src') {

                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t anjaligundavaram/cartservice:latest ."
                    }
                  }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker push anjaligundavaram/cartservice:latest "
                    }
                }
            }
        }
    }
}
