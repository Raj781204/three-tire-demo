pipeline {
    agent any

    environment {
        IMAGE_NAME = "dinesh781204/three-tier-app"
        TAG = "${BUILD_NUMBER}"
    }

    stages {

        // ✅ FIXED (no duplicate checkout issue)
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // ✅ INSTALL DEPENDENCIES
        stage('Install Dependencies') {
            steps {
                sh 'cd backend && npm install'
                sh 'cd frontend && npm install'
            }
        }

        // ✅ BUILD FRONTEND
        stage('Build') {
            steps {
                sh 'cd frontend && npm run build'
            }
        }

        // ✅ RUN TESTS
        stage('Test') {
            steps {
                sh 'cd backend && npm test'
            }
        }

        // ✅ SONARQUBE ANALYSIS
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }

        // ✅ QUALITY GATE
        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // ✅ DOCKER BUILD (FIXED PATH)
        stage('Docker Build') {
            steps {
                sh 'cd docker && docker-compose build'
            }
        }

        // ✅ DOCKER LOGIN + PUSH (SECURE)
        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    cd docker && docker-compose push
                    '''
                }
            }
        }

        // ✅ DEPLOY (FIXED PATH)
        stage('Deploy') {
            steps {
                sh 'cd docker && docker-compose down'
                sh 'cd docker && docker-compose up -d'
            }
        }

        // ✅ VERIFY
        stage('Verify') {
            steps {
                sh 'docker ps'
            }
        }
    }

    // ✅ CLEAN WORKSPACE
    post {
        always {
            cleanWs()
        }
    }
}
