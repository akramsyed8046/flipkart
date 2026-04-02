pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'JDK25'
    }

    environment {
        SONARQUBE_ENV = 'sq'
        DOCKER_IMAGE = "akramsyed8046/flipkart:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/akramsyed8046/flipkart.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        // 🔥 Sonar Added Here (from pipeline 1)
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Nexus') {
            steps {
                withMaven(
                    maven: 'maven3',
                    jdk: 'JDK25',
                    globalMavenSettingsConfig: 'settings.xml'
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

        stage('Deploy Container') {
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
