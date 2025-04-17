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
        stage('Build Docker Image') {
            steps {
                // Build Docker image
                script {
                    docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        // Log in to Docker Hub
                        sh "echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USER} --password-stdin"

                        // Push Docker images to Docker Hub
                        sh "docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }

    }
}
