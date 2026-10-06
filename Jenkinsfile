pipeline {

    agent any

    environment {

        // Docker Hub
        DOCKER_IMAGE = 'kaustubh10k/ekdantay'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials'

        // Nexus
        NEXUS_REGISTRY = 'localhost:8082'
        NEXUS_IMAGE = 'localhost:8082/ekdantay'
        NEXUS_CREDENTIALS = 'nexus-credentials'

        // AWS ECR
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '392211784456'
        ECR_REGISTRY = '392211784456.dkr.ecr.ap-south-1.amazonaws.com'
        ECR_REPOSITORY = 'ekdantay-ecr'
        ECR_IMAGE = '392211784456.dkr.ecr.ap-south-1.amazonaws.com/ekdantay-ecr'
        AWS_CREDENTIALS = 'aws-ecr-credentials'
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
                    echo "===== DOCKER BUILD ====="

                    docker build \
                        -f dockerfile \
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
                    echo "===== PUSHING TO DOCKER HUB ====="

                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Nexus Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${NEXUS_CREDENTIALS}",
                        usernameVariable: 'NEXUS_USERNAME',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASSWORD" | docker login ${NEXUS_REGISTRY} \
                            -u "$NEXUS_USERNAME" \
                            --password-stdin
                    '''
                }
            }
        }

        stage('Tag for Nexus') {
            steps {
                sh '''
                    echo "===== TAGGING IMAGE FOR NEXUS ====="

                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        ${NEXUS_IMAGE}:${BUILD_NUMBER}

                    docker tag ${DOCKER_IMAGE}:latest \
                        ${NEXUS_IMAGE}:latest
                '''
            }
        }

        stage('Push to Nexus') {
            steps {
                sh '''
                    echo "===== PUSHING TO NEXUS ====="

                    docker push ${NEXUS_IMAGE}:${BUILD_NUMBER}
                    docker push ${NEXUS_IMAGE}:latest
                '''
            }
        }

        stage('ECR Login') {
            steps {
                echo '===== LOGGING IN TO AWS ECR ====='

                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                     credentialsId: "${AWS_CREDENTIALS}"]
                ]) {
                    sh '''
                        aws ecr get-login-password \
                            --region ${AWS_REGION} | \
                        docker login \
                            --username AWS \
                            --password-stdin ${ECR_REGISTRY}
                    '''
                }
            }
        }

        stage('Tag for ECR') {
            steps {
                sh '''
                    echo "===== TAGGING IMAGE FOR ECR ====="

                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                        ${ECR_IMAGE}:${BUILD_NUMBER}

                    docker tag ${DOCKER_IMAGE}:latest \
                        ${ECR_IMAGE}:latest
                '''
            }
        }

        stage('Push to ECR') {
            steps {
                echo '===== PUSHING IMAGE TO AWS ECR ====='

                sh '''
                    docker push ${ECR_IMAGE}:${BUILD_NUMBER}
                    docker push ${ECR_IMAGE}:latest
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo ' EKDANTAY CI PIPELINE SUCCESSFUL'
            echo '======================================'

            echo "Docker Hub : ${DOCKER_IMAGE}:${BUILD_NUMBER}"
            echo "Nexus      : ${NEXUS_IMAGE}:${BUILD_NUMBER}"
            echo "AWS ECR    : ${ECR_IMAGE}:${BUILD_NUMBER}"
        }

        failure {
            echo '======================================'
            echo ' EKDANTAY CI PIPELINE FAILED'
            echo '======================================'
        }

        always {
            sh 'docker logout || true'
            sh 'docker logout ${NEXUS_REGISTRY} || true'
            sh 'docker logout ${ECR_REGISTRY} || true'
            sh 'docker image prune -f || true'
        }
    }
}