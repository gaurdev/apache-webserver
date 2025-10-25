pipeline {
    agent any
    environment {
        registry = "garvmanster/myapache"
        registryCredential = 'docker-credentials'
        dockerImage = ''
	REMOTE_USER = 'root'
        REMOTE_HOST = '192.168.17.138'
        REMOTE = "${REMOTE_USER}@${REMOTE_HOST}"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/gaurdev/apache-webserver.git', credentialsId: 'github-credentials'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }
        stage('Test Container') {
            steps {
                script {
                    sh "docker run -d --name apache-test -p 8888:80 ${registry}:${BUILD_NUMBER}"
                    sh "sleep 5"
                    sh "curl -f http://localhost:8888"
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
	stage('Deploy') {
            steps {
		sshagent(['ssh-key']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ${REMOTE} <<'EOF'
                docker pull ${registry}:${BUILD_NUMBER}
                docker rm -f apache-live || true
                docker run -d --restart=always --name apache-live -p 2080:80 ${registry}:${BUILD_NUMBER}
EOF
                '''
            }
        }
}
}
}
