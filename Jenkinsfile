pipeline {

    agent any

    tools {
        maven 'Maven-3.8.5'
    }

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
        
            environment {
                scannerHome = tool 'SonarScanner'
            }
        
            steps {
        
                withSonarQubeEnv('sonarqube') {
        
                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=spring-boot-mongo \
                    -Dsonar.projectName=spring-boot-mongo \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
        
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
                withCredentials([
                    usernamePassword(
                        credentialsId: 'mongodb-creds',
                        usernameVariable: 'MONGO_USER',
                        passwordVariable: 'MONGO_PASS'
                    )
                ]) {
                    sh '''
                        docker rm -f spring-boot-app || true
                        docker rm -f mongo || true
        
                        docker run -d \
                          --name mongo \
                          --network devops \
                          -p 27017:27017 \
                          -e MONGO_INITDB_ROOT_USERNAME=$MONGO_USER \
                          -e MONGO_INITDB_ROOT_PASSWORD=$MONGO_PASS \
                          mongo:7.0.9
        
                        sleep 20
        
                        docker run -d \
                          --name spring-boot-app \
                          --network devops \
                          -p 8085:8080 \
                          -e SPRING_DATA_MONGODB_URI="mongodb://$MONGO_USER:$MONGO_PASS@mongo:27017/admin" \
                          localhost:8082/docker-hosted/spring-boot-mongo:${BUILD_NUMBER}
                    '''
                }
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
