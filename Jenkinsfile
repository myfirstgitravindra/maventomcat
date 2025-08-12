pipeline {
    agent any
    
    environment {
        SONAR_TOKEN = credentials('sonarnew')  // Use 'sonarnew' to match your credential ID
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/myfirstgitravindra/maventomcat.git'
            }
        }
        
        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {  // Must match exact Jenkins config name
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=myproject \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://54.162.245.70:9000 \
                          -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }
    }
}
