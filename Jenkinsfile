
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT_ID = '428189921058'
        ECR_REPO = 'aws-cicd-demo'
        ECR_REGISTRY = '428189921058.dkr.ecr.us-east-1.amazonaws.com'
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

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-ecr-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        set +x
                        export AWS_ACCESS_KEY_ID="$AWS_ACCESS_KEY_ID"
                        export AWS_SECRET_ACCESS_KEY="$AWS_SECRET_ACCESS_KEY"
                        aws ecr get-login-password --region "$AWS_REGION" |
                        docker login --username AWS --password-stdin "$ECR_REGISTRY"
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    docker tag "$ECR_REPO:$IMAGE_TAG" "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                    docker push "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                '''
            }
        }
    }
}
