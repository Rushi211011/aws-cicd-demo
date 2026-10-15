
pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
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
                        set -e
                        export AWS_DEFAULT_REGION="$AWS_REGION"
                        aws ecr get-login-password --region "$AWS_REGION" |
                          docker login --username AWS --password-stdin "$ECR_REGISTRY"
                    '''
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                    set -e
                    docker tag "$ECR_REPO:$IMAGE_TAG" "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                    docker push "$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG"
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([string(
                    credentialsId: 'k8s-deployer-token',
                    variable: 'K8S_TOKEN'
                )]) {
                    sh '''
                        set -e
                        export KUBECONFIG="$WORKSPACE/.kubeconfig"

                        kubectl config set-cluster docker-desktop \
                          --server=https://kubernetes.docker.internal:6443 \
                          --certificate-authority=/tmp/k8s-ca.crt \
                          --embed-certs=true

                        kubectl config set-credentials jenkins-deployer \
                          --token="$K8S_TOKEN"

                        kubectl config set-context cicd-context \
                          --cluster=docker-desktop \
                          --user=jenkins-deployer \
                          --namespace=cicd

                        kubectl config use-context cicd-context

                        kubectl set image deployment/aws-cicd-demo \
                          aws-cicd-demo="$ECR_REGISTRY/$ECR_REPO:$IMAGE_TAG" \
                          -n cicd

                        kubectl rollout status deployment/aws-cicd-demo \
                          -n cicd --timeout=180s
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f "$WORKSPACE/.kubeconfig"'
        }
    }
}

                                          
                
            
     

        
           
