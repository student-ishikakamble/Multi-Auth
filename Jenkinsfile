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
                    credentialsId: 'github-pat-token',
                    url: 'https://github.com/student-ishikakamble/Multi-Auth.git'

            }

        }



        stage('Install Dependencies') {

            steps {

                sh '''
                npm install
                '''

            }

        }



        stage('Generate Prisma Client') {

            steps {

                sh '''
                npx prisma generate
                '''

            }

        }



        stage('Database Migration') {

            steps {

               withCredentials([

    string(credentialsId: 'multi-auth-db-url', variable: 'DATABASE_URL'),

    string(credentialsId: 'jwt-private-key', variable: 'JWT_PRIVATE_KEY'),

    string(credentialsId: 'jwt-public-key', variable: 'JWT_PUBLIC_KEY')

]) {


                    sh '''

                    export DATABASE_URL=$DATABASE_URL

                    echo "Running Prisma migration..."

                    npx prisma migrate deploy

                    '''

                }

            }

        }



        stage('Docker Build') {

            steps {

                sh '''

                docker build \
                -t ${IMAGE_NAME}:latest .

                '''

            }

        }



        stage('Deploy') {

    steps {

        withCredentials([

            string(
                credentialsId: 'multi-auth-db-url',
                variable: 'DATABASE_URL'
            ),

            string(
                credentialsId: 'jwt-private-key',
                variable: 'JWT_PRIVATE_KEY'
            ),

            string(
                credentialsId: 'jwt-public-key',
                variable: 'JWT_PUBLIC_KEY'
            )

        ])  {


                    sh '''

                    echo "Stopping old container..."

                    docker stop ${APP_NAME} || true

                    docker rm ${APP_NAME} || true



                    echo "Starting new container..."

                docker run -d \
--name ${APP_NAME} \
-p ${PORT}:5000 \
-e DATABASE_URL="$DATABASE_URL" \
-e JWT_PRIVATE_KEY="$JWT_PRIVATE_KEY" \
-e JWT_PUBLIC_KEY="$JWT_PUBLIC_KEY" \
-e JWT_ACCESS_TOKEN_EXPIRE=900 \
-e JWT_REFRESH_TOKEN_EXPIRE=604800 \
${IMAGE_NAME}:latest


                    '''


                }


            }


        }




        stage('Health Check') {


            steps {


                sh '''


                echo "Waiting for application..."

                sleep 10



                curl --fail \
                http://localhost:${PORT}/health



                echo "Health check passed"


                '''


            }


        }



    }



    post {


        success {


            echo '''

            =================================
            Multi Auth Deployment Successful
            =================================

            '''


        }



        failure {


            echo '''

            =================================
            Deployment Failed
            Rollback required
            =================================

            '''


        }


    }


}
