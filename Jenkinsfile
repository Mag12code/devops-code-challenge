pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-2'
        AWS_ACCOUNT_ID = '340752806399'
        ECR_REPO = 'my-application-repo'
        IMAGE_TAG_FRONTEND = 'frontend-2'
        IMAGE_TAG_BACKEND = 'backend-2'
        ECS_CLUSTER_NAME = 'my-cluster'
        AWS_ACCESS_KEY = credentials('aws-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Configure AWS') {
            steps {
                sh '''
                    aws configure set aws_access_key_id $AWS_ACCESS_KEY_USR
                    aws configure set aws_secret_access_key $AWS_ACCESS_KEY_PSW
                    aws configure set region ${AWS_REGION}
                    aws configure set output json
                '''
            }
        }
        
        stage('Build Docker Images') {
            steps {
                sh '''
                    docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_FRONTEND} frontend/
                    docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_BACKEND} backend/
                '''
            }
        }
        
        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_FRONTEND}
                    docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_BACKEND}
                '''
            }
        }
        
        stage('Deploy to ECS') {
            steps {
                sh '''
                    aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service frontend-service --force-new-deployment --region ${AWS_REGION}
                    aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service backend-service --force-new-deployment --region ${AWS_REGION}
                '''
            }
        }
    }
    
    post {
        always {
            sh '''
                docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_FRONTEND} || true
                docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG_BACKEND} || true
            '''
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
