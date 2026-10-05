pipeline {

    agent any

    environment {
        DOCKER_IMAGE = 'kaustubh10k/ekdantay'

        NEXUS_REGISTRY = '13.201.22.239:8082'
        NEXUS_IMAGE = '13.201.22.239:8082/ekdantay'

        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'
        NEXUS_CREDENTIALS = 'nexus-credentials'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out Ekdantay source code...'

                git branch: 'main',
                    url: 'https://github.com/kaustubha10/Ekdantay.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh '''
                    export MAVEN_HOME=/opt/apache-maven-3.10.0
                    export PATH=$MAVEN_HOME/bin:$PATH

                    echo "===== MAVEN VERSION ====="
                    mvn -version

                    echo "===== BUILDING APPLICATION ====="
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        -t ${DOCKER_IMAGE}:latest .
                '''
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                sh '''
                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

       stage('Nexus Login') {
           steps {
               withCredentials([usernamePassword(
                   credentialsId: 'dockerhub-credentials',
                   usernameVariable: 'NEXUS_USERNAME',
                   passwordVariable: 'NEXUS_PASSWORD'
               )]) {
                   sh '''
                       echo "$NEXUS_PASSWORD" | docker login localhost:8082 \
                           -u "$NEXUS_USERNAME" \
                           --password-stdin
                   '''
               }
           }
       }

       stage('Tag for Nexus') {
           steps {
               sh '''
                   docker tag kaustubh10k/ekdantay:${BUILD_NUMBER} \
                       localhost:8082/ekdantay:${BUILD_NUMBER}

                   docker tag kaustubh10k/ekdantay:latest \
                       localhost:8082/ekdantay:latest
               '''
           }
       }

       stage('Push to Nexus') {
           steps {
               sh '''
                   docker push localhost:8082/ekdantay:${BUILD_NUMBER}
                   docker push localhost:8082/ekdantay:latest
               '''
           }
       }

    post {

        success {
            echo '======================================'
            echo ' EKDANTAY CI PIPELINE SUCCESSFUL'
            echo '======================================'
            echo "Docker Hub : ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo "Nexus      : ${NEXUS_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo '======================================'
            echo ' EKDANTAY CI PIPELINE FAILED'
            echo '======================================'
        }

        always {
            sh 'docker logout || true'
            sh 'docker image prune -f || true'
        }
    }
}