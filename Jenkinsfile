pipeline {

    agent any


    environment {

        APP_NAME = "spring-boot-mongo"

        IMAGE_NAME = "localhost:8082/${APP_NAME}"

        SONAR_HOME = tool 'SonarScanner'

    }


    stages {


        stage('Checkout') {

            steps {

                git(
                    branch: 'main',
                    credentialsId: 'github-creds',
                    url: 'https://github.com/Mshabirhussain/spring-boot-mongo-docker-kkfunda.git'
                )

            }
        }



        stage('Maven Build') {

            steps {

                sh '''
                mvn clean package -DskipTests
                '''

            }
        }



        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''

                    mvn sonar:sonar \
                    -Dsonar.projectKey=${APP_NAME} \
                    -Dsonar.host.url=http://sonarqube:9000 \
                    -Dsonar.login=$SONAR_TOKEN

                    '''

                }

            }
        }



        stage('Build Docker Image') {

            steps {

                sh '''

                docker build \
                -t ${IMAGE_NAME}:${BUILD_NUMBER} .

                '''

            }

        }



        stage('Push Image To Nexus') {

            steps {


                withCredentials([
                    usernamePassword(
                    credentialsId:'nexus-docker-creds',
                    usernameVariable:'NEXUS_USER',
                    passwordVariable:'NEXUS_PASS'
                    )
                ]){


                sh '''

                echo $NEXUS_PASS | docker login localhost:8082 \
                -u $NEXUS_USER \
                --password-stdin


                docker push \
                ${IMAGE_NAME}:${BUILD_NUMBER}


                '''

                }

            }

        }



        stage('Deploy Container') {


            steps {


                sh '''

                docker stop ${APP_NAME} || true

                docker rm ${APP_NAME} || true


                docker run -d \
                --name ${APP_NAME} \
                -p 8085:8080 \
                ${IMAGE_NAME}:${BUILD_NUMBER}


                '''

            }

        }


    }


    post {


        success {

            echo "Deployment completed successfully"

        }


        failure {

            echo "Pipeline failed"

        }

    }

}
