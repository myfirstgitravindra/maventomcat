pipeline {
    agent any

    environment {
        // Customize these values
        DOCKER_IMAGE = "ravindra806/myapp:${BUILD_NUMBER}" // Use your Docker Hub username
        DOCKER_REGISTRY = "docker.io"
        SONAR_HOST_URL = "http://54.162.245.70:9000"
        PROJECT_KEY = "myproject"
    }

    stages {
        /* Stage 1: Git Checkout */
        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        /* Stage 2: SonarQube Analysis */
        stage('Code Quality Scan') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            environment {
                SONAR_TOKEN = credentials('sonarnew')
            }
            steps {
                sh """
                    sonar-scanner \
                    -Dsonar.projectKey=${PROJECT_KEY} \
                    -Dsonar.sources=. \
                    -Dsonar.host.url=${SONAR_HOST_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                """
            }
        }

        /* Stage 3: Quality Gate */
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* Stage 4: Docker Build */
        stage('Docker Build') {
            agent {
                docker {
                    image 'docker:20.10-dind'
                    args '--privileged'
                }
            }
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        /* Stage 5: Security Scan */
        stage('Vulnerability Scan') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                    args '--entrypoint='
                }
            }
            steps {
                sh """
                    trivy image \
                    --severity CRITICAL,HIGH \
                    --exit-code 1 \
                    --ignore-unfixed \
                    ${DOCKER_IMAGE}
                """
            }
        }

        /* Stage 6: Docker Push */
        stage('Push to Registry') {
            agent {
                docker {
                    image 'docker:20.10-cli'
                }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', // Matches your credential ID
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
            cleanWs()
        }
        failure {
            emailext (
                subject: "FAILED: Pipeline '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'your-email@example.com' // Replace with your email
            )
        }
    }
}
