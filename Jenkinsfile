pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        AWS_ACCOUNT_ID = '099771438874'
        ECR_REPOSITORY = 'tech-challenge-2'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER = 'tech-challenge-2-cluster'
    }

    stages {

        stage('Verify Tools') {
            steps {
                sh '''
                    git --version
                    docker --version
                    aws --version
                    kubectl version --client
                    helm version
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${ECR_REPOSITORY}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} \
                    | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                '''
            }
        }

        stage('Tag and Push Image') {
            steps {
                sh '''
                    docker tag \
                      ${ECR_REPOSITORY}:${IMAGE_TAG} \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}

                    docker push \
                      ${ECR_REGISTRY}/${ECR_REPOSITORY}:${IMAGE_TAG}
                '''
            }
        }

        stage('Connect to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}

                    kubectl get nodes
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    helm upgrade --install tech-challenge-2 ./helm/tech-challenge-2 \
                      --set image.repository=${ECR_REGISTRY}/${ECR_REPOSITORY} \
                      --set image.tag=${IMAGE_TAG}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl rollout status deployment/tech-challenge-2
                    kubectl get pods
                    kubectl get service
                    kubectl get ingress
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD deployment completed successfully.'
        }

        failure {
            echo 'CI/CD deployment failed. Check the stage logs.'
        }
    }
}