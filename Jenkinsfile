pipeline {
    agent any
    environment {
        registry = "garvmanster/myapache"
        registryCredential = 'docker-credentials'
        dockerImage = ''
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/gaurdev/apache-webserver.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = podman.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }
        stage('Test Container') {
            steps {
                script {
                    sh "docker run -d --name apache-test -p 8080:80 ${registry}:${BUILD_NUMBER}"
                    sh "sleep 5"
                    sh "curl -f http://localhost:8080"
                    sh "docker rm -f apache-test"
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                sh "docker rmi ${registry}:${BUILD_NUMBER}"
            }
        }
    }
}

