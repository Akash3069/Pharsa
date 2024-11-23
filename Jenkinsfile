pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Akash3069/Novaria.git'
            }
        }
     stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar-token") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'Dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build -t frontend ./frontend"
                       sh "docker build -t backend ./backend"
                       sh "docker tag frontend akash550/front-end:v1 "
                       sh "docker push akash550/front-end:v1  "
                       sh "docker tag backend akash550/back-end:v1 "
                       sh "docker push akash550/back-end:v1  "

                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image akash550/front-end:v1 > trivyimage.txt" 
                sh "trivy image akash550/back-end:v1 > trivyimage.txt" 

            }
        }    

    }
  
}