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

        // ✅ Debug (keep this for now)
        stage('Debug Files') {
            steps {
                sh 'pwd'
                sh 'ls -l'
                sh 'ls -l backend || true'
                sh 'ls -l frontend || true'
            }
        }

        // ✅ Backend Dependencies
        stage('Install Backend Dependencies') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }

        // ✅ Docker Build (ONLY ONCE)
        stage('Docker Build') {
            steps {
                dir('docker') {
                    sh 'docker-compose build'
                }
            }
        }

        // ✅ Login & Push (NO BUILD HERE)
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKERHUB_CREDENTIALS}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    cd docker
                    docker-compose push
                    '''
                }
            }
        }

        // ✅ Deploy
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
