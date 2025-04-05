pipeline {
    agent any
    
   tools{
        jdk 'jdk17'
        nodejs 'node16'
        
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dheeraj7444/fullstack-bank.git'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
               dependencyCheck additionalArguments: '--scan ./app/backend --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        } 
        stage('Trive FS Scan') {
            steps {
              sh "trivy fs ."
            }
        }
        stage('SONARQUBE ANALYSIS') {
            steps {
             withSonarQubeEnv('sonar') {
             sh " $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Bank -Dsonar.projectKey=Bank "
   
               }  
            }
        }    
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }    
        }
        stage('Backend') {
            steps {
                dir('app/backend') {
                    sh "npm install"
                }
            }
        }
        
        stage('frontend') {
            steps {
                dir('app/frontend') {
                    sh "npm install"
                }
            }
        }
        stage('Deploy to Conatiner') {
            steps {
               sh 'cd app && docker compose build'
            }
        }
        stage('Docker Puss') {
            steps {
               withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
              sh """
                 echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                  docker tag dheeraj7444/ng_frontend:latest dheeraj7444/ng_frontend:$BUILD_ID
                  docker tag dheeraj7444/ng_backend:latest dheeraj7444/ng_backend:$BUILD_ID
                  docker push dheeraj7444/ng_frontend:$BUILD_ID
                  docker push dheeraj7444/ng_backend:$BUILD_ID
                  docker rmi -f dheeraj7444/ng_frontend:latest dheeraj7444/ng_backend:latest dheeraj7444/ng_frontend:$BUILD_ID dheeraj7444/ng_backend:$BUILD_ID
                 """
               } 
            }
        }
    }
}
