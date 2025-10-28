pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9'
    }
    
    environment {
        DOCKER_IMAGE = 'java-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        JAVA_HOME = 'C:\\Program Files\\Java\\jdk-17'
        PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from repository...'
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building Java application with Maven...'
                bat 'mvn clean compile package'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running tests...'
                bat 'mvn test'
            }
        }
        
        stage('Verify Build') {
            steps {
                echo 'Verifying JAR file contents...'
                bat 'dir target\\*.jar'
                bat 'jar tf target\\java-app-1.0-SNAPSHOT.jar'
            }
        }
        
        stage('Execute Application') {
            steps {
                echo 'Executing Java application...'
                bat 'cd target\\classes && java com.example.App'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                script {
                    bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    bat "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
                }
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        bat "docker login -u %DOCKER_USER% -p %DOCKER_PASS%"
                        bat "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} %DOCKER_USER%/${DOCKER_IMAGE}:${DOCKER_TAG}"
                        bat "docker push %DOCKER_USER%/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            echo 'Build artifacts:'
            bat 'dir target\\*.jar'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}