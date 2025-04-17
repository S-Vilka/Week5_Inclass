pipeline {
    agent any
    tools {
        maven 'Maven3'
        dockerTool 'DOCKER'
    }
    environment {
        // Define Docker Hub credentials ID.
        DOCKERHUB_CREDENTIALS_ID = 'Docker_Hub'
        // Define Docker Hub repository name
        DOCKERHUB_REPO = 'sergeivilka/tripcostcalculator'
        // Define Docker image tag
        DOCKER_IMAGE_TAG = 'latest_v1'
        SONARQUBE_SERVER = 'SonarQubeServer'  // The name of the SonarQube server configured in Jenkins
        SONAR_TOKEN = 'squ_32b89a413a0567fe7f3471f0bddecee1531c938a' // Store the token securely
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/S-Vilka/Week5_Inclass.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh """
                        /Users/sergeivilka/sonar-scanner-7.1.0.4889-macosx-aarch64/bin/sonar-scanner \
                        -Dsonar.projectKey=devops-demo \
                        -Dsonar.sources=src \
                        -Dsonar.projectName=DevOps-Demo \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${env.SONAR_TOKEN} \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
        stage('Docker Login') {
                            steps {
                                script {
                                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID,
                                                                     usernameVariable: 'DOCKERHUB_USER',
                                                                     passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                                        sh '''
                                            echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USER" --password-stdin
                                        '''
                                    }
                                }
                            }
                        }
                        stage('Build Docker Image') {
                             steps {
                                 script {
                                     sh """
                                         docker buildx use mybuilder || docker buildx create --use --name mybuilder
                                         docker buildx build --platform=linux/amd64 -t ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} --load .
                                     """
                                 }
                             }
                         }

                        stage('Push Docker Image to Docker Hub') {
                            steps {
                                script {
                                    sh "docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
                                }
                            }
                        }

            }
        }