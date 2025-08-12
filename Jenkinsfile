pipeline {
    agent none  // We'll define agents per stage

    environment {
        // Customize these values
        DOCKER_IMAGE = "yourdockerhub/myapp:${1}"
        DOCKER_REGISTRY = "docker.io"
        SONAR_HOST_URL = "http://54.162.245.70:9000"
        PROJECT_KEY = "myproject"
    }

    stages {
        /* Stage 1: Git Checkout */
        stage('Git Checkout') {
            agent any
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/myfirstgitravindra/maventomcat.git'
                    ]]
                ])
            }
        }

        /* Stage 2: SonarQube Analysis */
        stage('Code Quality Scan') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                    reuseNode true
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
                    args '--privileged --network host'
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
                    --no-progress \
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
            environment {
                DOCKER_CREDS = credentials('dockerhub-creds')
            }
            steps {
                sh """
                    docker login \
                    -u ${DOCKER_CREDS_USR} \
                    -p ${DOCKER_CREDS_PSW} \
                    ${DOCKER_REGISTRY}
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        /* Stage 7: Deploy (Example) */
        stage('Deploy to Staging') {
            when {
                branch 'main'
            }
            agent any
            steps {
                echo "Deploying ${DOCKER_IMAGE} to staging environment"
                // Add your deployment commands here
                // Example: kubectl apply -f k8s/deployment.yaml
            }
        }
    }

    post {
        always {
            echo "Pipeline execution completed"
            cleanWs()  // Clean workspace
        }
        success {
            slackSend(
                color: 'good',
                message: "SUCCESS: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "FAILED: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
            emailext (
                subject: "FAILED: Pipeline '${env.JOB_NAME}' (${env.BUILD_NUMBER})",
                body: "Check console output at ${env.BUILD_URL}",
                to: 'devops-team@example.com'
            )
        }
    }
}
