pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-2'
        AWS_ACCOUNT_ID = '340752806399'
        ECR_REPO = 'my-application-repo'
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECS_CLUSTER_NAME = 'my-cluster'
        AWS_ACCESS_KEY = credentials('aws-credentials')
    }
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    try {
                        checkout scm
                    } catch (Exception e) {
                        error "Failed to checkout code: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Verify Environment') {
            steps {
                sh '''
                    echo "=== System Information ==="
                    uname -a
                    echo "\n=== Docker Version ==="
                    docker --version
                    echo "\n=== AWS CLI Version ==="
                    aws --version
                    
                    echo "\n=== Current Directory ==="
                    pwd
                    echo "\n=== Directory Contents ==="
                    ls -la
                    
                    echo "\n=== Frontend Directory ==="
                    ls -la frontend/
                    echo "\n=== Frontend Dockerfile ==="
                    cat frontend/Dockerfile
                    
                    echo "\n=== Backend Directory ==="
                    ls -la backend/
                    echo "\n=== Backend Dockerfile ==="
                    cat backend/Dockerfile
                '''
            }
        }
        
        stage('Configure AWS') {
            steps {
                script {
                    try {
                        sh '''
                            aws configure set aws_access_key_id $AWS_ACCESS_KEY_USR
                            aws configure set aws_secret_access_key $AWS_ACCESS_KEY_PSW
                            aws configure set region ${AWS_REGION}
                            aws configure set output json
                            
                            # Verify AWS Configuration
                            aws sts get-caller-identity
                        '''
                    } catch (Exception e) {
                        error "Failed to configure AWS: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Create ECR Repository') {
            steps {
                script {
                    try {
                        sh '''
                            # Create ECR repository if it doesn't exist
                            aws ecr describe-repositories --repository-names ${ECR_REPO} || \
                            aws ecr create-repository --repository-name ${ECR_REPO} --region ${AWS_REGION}
                        '''
                    } catch (Exception e) {
                        echo "Repository might already exist: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    try {
                        sh '''
                            # Build Frontend
                            echo "Building frontend image..."
                            cd frontend
                            docker build --no-cache -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:frontend-${IMAGE_TAG} .
                            
                            # Build Backend
                            echo "Building backend image..."
                            cd ../backend
                            docker build --no-cache -t ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:backend-${IMAGE_TAG} .
                        '''
                    } catch (Exception e) {
                        error "Failed to build Docker images: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Push to ECR') {
            steps {
                script {
                    try {
                        sh '''
                            # Login to ECR
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                            
                            # Push images
                            echo "Pushing frontend image..."
                            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:frontend-${IMAGE_TAG}
                            
                            echo "Pushing backend image..."
                            docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:backend-${IMAGE_TAG}
                        '''
                    } catch (Exception e) {
                        error "Failed to push images to ECR: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Deploy to ECS') {
            steps {
                script {
                    try {
                        sh '''
                            # Update ECS services
                            aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service frontend-service --force-new-deployment --region ${AWS_REGION}
                            aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service backend-service --force-new-deployment --region ${AWS_REGION}
                        '''
                    } catch (Exception e) {
                        error "Failed to deploy to ECS: ${e.getMessage()}"
                    }
                }
            }
        }
    }
    
    post {
        always {
            script {
                try {
                    sh '''
                        echo "Cleaning up Docker images..."
                        docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:frontend-${IMAGE_TAG} || true
                        docker rmi ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:backend-${IMAGE_TAG} || true
                    '''
                } catch (Exception e) {
                    echo "Warning: Cleanup failed: ${e.getMessage()}"
                }
            }
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs above for details.'
        }
    }
}
