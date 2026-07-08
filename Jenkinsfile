pipeline {
    agent any

    environment {
        APP_NAME = "multi-auth-container"
        IMAGE_NAME = "multi-auth-backend"
        PORT = "5000"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                url: 'https://github.com/student-ishikakamble/Multi-Auth.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Generate Prisma Client') {
            steps {
                sh 'npx prisma generate'
            }
        }

        stage('Database Migration') {
            steps {
                sh 'npx prisma migrate deploy'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                docker stop ${APP_NAME} || true
                docker rm ${APP_NAME} || true

                docker run -d \
                --name ${APP_NAME} \
                -p ${PORT}:5000 \
                --env-file .env \
                ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 10

                curl --fail http://localhost:${PORT}/health
                '''
            }
        }
    }

    post {

        failure {
            echo "Deployment failed - rollback required"
        }

        success {
            echo "Multi Auth deployment successful"
        }
    }
}
