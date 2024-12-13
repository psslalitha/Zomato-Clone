pipeline {
    agent any
    tools{
        jdk 'java17'
        nodejs 'node16'
    }
    environment{
        SCANNER_HOME=tool 'sonarqube'
    }
    

    stages {
         stage('clean-ws') {
            steps {
                cleanWs()
            }
        }
        stage('git-checkout') {
            steps {
                git 'https://github.com/psslalitha/Zomato-Clone.git'
            }
        }
        stage('sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zp \
                    -Dsonar.projectKey=zp'''
                }
            }
        }
        stage('quality-gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 's-id' 
                }
            }
        }
        stage('dependency'){
            steps{
                sh 'npm install'
            }
        }
        
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dp-check'
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
                   withDockerRegistry(credentialsId: 'd-id', toolName: 'docker'){   
                       sh "docker build -t zomato ."
                       sh "docker tag zomato srilalithac/zomato:latest "
                       sh "docker push srilalithac/zomato:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image srilalithac/zomato:latest > trivy.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name zomato -p 3000:3000 srilalithac/zomato:latest'
            }
        }

    }
}
