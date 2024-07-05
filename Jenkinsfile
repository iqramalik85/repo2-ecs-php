pipeline {
    agent any
    environment {
        ECR_REPO = '736116236436.dkr.ecr.us-east-1.amazonaws.com' // ECR repository URL
        IMAGE_NAME = 'my-img2' // Desired image name
        AWS_REGION = 'us-east-1' // Correct AWS region
        ECS_CLUSTER_NAME = 'ecs-php' // Replace with your ECS cluster name
        ECS_SERVICE_NAME = 'myecs-service' // Replace with your ECS service name
    }
    parameters {
        string(name: 'ENVIRONMENT_NAME', defaultValue: 'my-img2', description: 'Environment name')
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker image tag')
        string(name: 'ECR_URL', defaultValue: '736116236436.dkr.ecr.us-east-1.amazonaws.com', description: 'ECR repository URL')
    }
    stages {
        stage('Clone Repository') {
            steps {
                deleteDir()
                git credentialsId: 'GITHUB-CREDENTIALS', url: 'https://github.com/iqramalik85/repo2-ecs-php.git', branch: 'main'
            }
        }
        stage('Create new ECR repository') {
            environment {
                AWS_DEFAULT_REGION = 'us-east-1'
            }
            steps {
                script {
                    try {
                        sh "aws ecr create-repository --repository-name ${params.ENVIRONMENT_NAME}"
                    } catch (Exception e) {
                        echo "Repository ${params.ENVIRONMENT_NAME} already exists. Skipping repository creation."
                    }
                }
            }
        }
        stage('Push new image to ECR') {
            steps {
                withCredentials([
                    string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh "aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${params.ECR_URL}"
                    sh "docker build -t ${params.ECR_URL}/${params.ENVIRONMENT_NAME}:${params.DOCKER_TAG} ."
                    sh "docker push ${params.ECR_URL}/${params.ENVIRONMENT_NAME}:${params.DOCKER_TAG}"
                }
            }
        }
        stage('Ensure ECS Cluster Exists') {
            steps {
                script {
                    try {
                        sh "aws ecs describe-clusters --clusters ${env.ECS_CLUSTER_NAME}"
                    } catch (Exception e) {
                        echo "ECS cluster ${env.ECS_CLUSTER_NAME} does not exist. Creating it now."
                        sh "aws ecs create-cluster --cluster-name ${env.ECS_CLUSTER_NAME}"
                    }
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                script {
                    sh """
                    aws ecs update-service --cluster ${env.ECS_CLUSTER_NAME} --service ${env.ECS_SERVICE_NAME} --force-new-deployment
                    """
                }
            }
        }
    }
}
