pipeline {
    agent any
    environment {
        FRONTEND_IMAGE = "streaming-app-frontend"
        HELLO_IMAGE = "streaming-app-hello"
        PROFILE_IMAGE = "streaming-app-profile"
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('AWS ECR Login & Push') {
            steps {
                // Bind AWS Credentials, Account ID, and Default Region from Jenkins Credentials vault
                withCredentials([
                    string(credentialsId: 'aws-account-id', variable: 'AWS_ACCOUNT_ID'),
                    string(credentialsId: 'aws-default-region', variable: 'AWS_DEFAULT_REGION'),
                    [$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']
                ]) {
                    script {
                        def ecrRegistry = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        
                        // Log in to ECR
                        sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${ecrRegistry}"
                        
                        // 1. Build and Push Frontend
                        sh "docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} -f docker/frontend.Dockerfile ."
                        sh "docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                        sh "docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${FRONTEND_IMAGE}:latest"
                        sh "docker push ${ecrRegistry}/${FRONTEND_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${ecrRegistry}/${FRONTEND_IMAGE}:latest"
                        
                        // 2. Build and Push Hello Service
                        sh "docker build -t ${HELLO_IMAGE}:${BUILD_NUMBER} -f docker/hello-service.Dockerfile ."
                        sh "docker tag ${HELLO_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${HELLO_IMAGE}:${BUILD_NUMBER}"
                        sh "docker tag ${HELLO_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${HELLO_IMAGE}:latest"
                        sh "docker push ${ecrRegistry}/${HELLO_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${ecrRegistry}/${HELLO_IMAGE}:latest"
                        
                        // 3. Build and Push Profile Service
                        sh "docker build -t ${PROFILE_IMAGE}:${BUILD_NUMBER} -f docker/profile-service.Dockerfile ."
                        sh "docker tag ${PROFILE_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${PROFILE_IMAGE}:${BUILD_NUMBER}"
                        sh "docker tag ${PROFILE_IMAGE}:${BUILD_NUMBER} ${ecrRegistry}/${PROFILE_IMAGE}:latest"
                        sh "docker push ${ecrRegistry}/${PROFILE_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${ecrRegistry}/${PROFILE_IMAGE}:latest"
                    }
                }
            }
        }
    }
    post {
        always {
            // Clean up unused docker images to save space on Jenkins builder
            sh "docker image prune -f"
        }
    }
}
