pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'dockerhub'
    }

    stages {

        // ✅ Checkout
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ✅ Debug (VERY IMPORTANT)
        stage('Debug Files') {
            steps {
                sh 'pwd'
                sh 'ls -l'
                sh 'ls -l backend'
                sh 'ls -l frontend'
            }
        }

        // ✅ Install Backend Dependencies ONLY
        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        // ✅ Docker Build
        stage('Docker Build') {
            steps {
                dir('docker') {
                    sh 'docker-compose build'
                }
            }
        }

        // ✅ Docker Login & Push (OPTIONAL but correct way)
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    
                    docker tag project_backend $DOCKER_USER/backend:v1
                    docker tag project_frontend $DOCKER_USER/frontend:v1
                    
                    docker push $DOCKER_USER/backend:v1
                    docker push $DOCKER_USER/frontend:v1
                    '''
                }
            }
        }

        // ✅ Deploy Containers
        stage('Deploy') {
            steps {
                dir('docker') {
                    sh 'docker-compose down || true'
                    sh 'docker-compose up -d'
                }
            }
        }

        // ✅ Verify
        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
