pipeline {
    agent any
    
    environment {
        SONAR_TOKEN = credentials('sonarqube') // Jenkins credentials ID for Sonar token
    }
    
    stages {
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/myfirstgitravindra/maventomcat.git'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=myproject \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://54.162.245.70:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
    }
}
