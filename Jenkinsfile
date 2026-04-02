pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = "akramsyed8046/flipkart:latest"
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

        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    maven: 'maven3',
                    jdk: 'jdk17',
                    globalMavenSettingsConfig: 'settings.xml',
                    traceability: true
                ) {
                    sh 'mvn clean deploy -DskipTests'
                }
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
                sh '''
                docker rm -f flipkart-app || true
                docker run -d --name flipkart-app -p 8999:8080 akramsyed8046/flipkart:latest
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
