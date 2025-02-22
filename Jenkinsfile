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
                git branch: 'main', url: 'https://github.com/Kingslekez/a-reddit-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('Sonar-Server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=RedditCloneCI \
                    -Dsonar.projectKey=RedditCloneCI '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token' 
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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
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
                   withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                       sh "docker build -t redditapp ."
                       sh "docker tag redditapp kingslekez/redditapp:latest "
                       sh "docker push kingslekez/redditapp:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image kingslekez/redditapp:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name redditapp -p 3000:3000 kingslekez/redditapp:latest'
            }
        }

    }
}
