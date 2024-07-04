pipeline {
    agent any
    environment {
        AWS_REGION = '<us-east-1>'
        ECR_REPO = '736116236436.dkr.ecr.us-east-1.amazonaws.com/php-app-image'
        ECS_CLUSTER = 'ecs-php'
        ECS_SERVICE = 'php-service'
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/iqramalik85/repo2-ecs-php.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("php-dummy-app")
                }
            }
        }
        stage('Login to Amazon ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                    }
                }
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    dockerImage.push('latest')
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                script {
                    sh """
                    aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment
                    """
                }
            }
        }
    }
}
