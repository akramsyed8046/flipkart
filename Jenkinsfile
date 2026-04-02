pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "akramsyed8046/flipkart:latest"
        DOCKER_CREDENTIALS = credentials('docker-hub') // must match Jenkins credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/akramsyed8046/flipkart.git'
            }
        }

        stage('Build WAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

         stage('JENKINS TO NEXUS') {
            steps {
              withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'jkd17', maven: 'maven3', traceability: true) {
             sh 'mvn deploy'
             }
         }


        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub', url: 'https://index.docker.io/v1/']) {
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Deploy Local Container') {
            steps {
                sh """
                docker rm -f flipkart-app || true
                docker run -d --name flipkart-app -p 8999:8080 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
