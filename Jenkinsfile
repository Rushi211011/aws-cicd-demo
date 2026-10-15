
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = 'aws-cicd-demo'
        AWS_ACCOUNT_ID = '428189921058'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $ECR_REPO:$IMAGE_TAG .'
            }
        }

        stage('Pipeline Ready') {
            steps {
                echo 'Source checkout and Docker build stages are configured.'
                echo 'ECR push and Kubernetes deployment will be configured next.'
            }
        }
    }
}
