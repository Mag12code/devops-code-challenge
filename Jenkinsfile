pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        AWS_ACCOUNT_ID = '340752806399'
        ECR_REPOSITORY = 'my-application-repo'
        FRONTEND_SERVICE_NAME = 'frontend-service'
        BACKEND_SERVICE_NAME = 'backend-service'
        ECS_CLUSTER_NAME = 'my-cluster'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Configure AWS') {
            steps {
                withCredentials([[
                    $class: 'AWSCredentialsBinding',
                    credentialsId: '637da2d6-d25c-4c10-b540-1bac7db21704',
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                    sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                    sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                    sh 'aws configure set region $AWS_DEFAULT_REGION'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh '''
                        docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:frontend-${BUILD_NUMBER} ./frontend
                        docker build -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:backend-${BUILD_NUMBER} ./backend
                    '''
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
                    sh '''
                        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:frontend-${BUILD_NUMBER}
                        docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:backend-${BUILD_NUMBER}
                    '''
                }
            }
        }
    }

    post {
        always {
            sh '''
                docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:frontend-${BUILD_NUMBER} || true
                docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:backend-${BUILD_NUMBER} || true
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
