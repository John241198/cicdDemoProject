pipeline{
    agent any
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Checkout from Git'){
            steps{
                git  'https://github.com/John241198/Multibranch-Pipeline.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=cicd \
                    -Dsonar.projectKey=cicd '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.json"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t cicd ."
                       sh "docker tag cicd jsnov24/cicd:latest "
                       sh "docker push jsnov24/cicd:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image jsnov24/cicd:latest > trivy.json"
            }
        }
        stage ("Remove container") {
            steps{
                sh "docker stop cicd | true"
                sh "docker rm cicd | true"
             }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name cicd -p 8010:80 jsnov24/cicd:latest'
            }
        }
    }
}
