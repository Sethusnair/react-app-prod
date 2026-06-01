pipeline {
    agent {
        docker {
            image 'node:20'
        }
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = 'react-app-prod'
        IMAGE_TAG = "${BUILD_NUMBER}"
        EC2_HOST = '54.92.254.91'
        EC2_USER = 'ubuntu'
    }

    stages {
        
        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'CI=true npm test -- --watchAll=false'
            }
        }



        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $ECR_REPO:$IMAGE_TAG .
                """
            }
        }

        stage('Login to ECR') {
            steps {
                sh 'aws ecr get-login-password --region us-east-1 | sudo docker login --username AWS --password-stdin 349551689958.dkr.ecr.us-east-1.amazonaws.com'
            }
        }

        stage('Tag Image') {
            steps {
                sh """
                docker tag $ECR_REPO:$IMAGE_TAG \
                349551689958.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG

                docker tag $ECR_REPO:$IMAGE_TAG \
                349551689958.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push 349551689958.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:$IMAGE_TAG
                docker push 349551689958.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:latest
                """
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-key']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no $EC2_USER@$EC2_HOST '
                        cd /opt/app &&
                        docker compose pull &&
                        docker compose up -d
                    '
                    """
                }
            }
        }
    }
}