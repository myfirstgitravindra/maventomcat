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
        
       stage('SonarQube Scan') {
            steps {
                // Ensure 'sonarqube' matches the exact name in Jenkins config (case-sensitive)
                withSonarQubeEnv('sonarqube') {  
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=myproject \  // Replace with your project key
                          -Dsonar.sources=. \             // Scan current directory
                          -Dsonar.host.url=http://54.162.245.70:9000 \  // From your config
                          -Dsonar.login=${SONAR_TOKEN}    // Uses the token securely
                    """
                }
            }
        }
    }
}
