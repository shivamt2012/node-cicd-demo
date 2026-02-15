pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "590183997695"
        REPO_NAME = "node-cicd-demo"
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPO_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
        TARGET_HOST = "10.0.10.252"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $REPO_NAME:$IMAGE_TAG .
                docker tag $REPO_NAME:$IMAGE_TAG $ECR_URI:$IMAGE_TAG
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_URI:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to Target EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@$TARGET_HOST "
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com &&
                    docker pull $ECR_URI:$IMAGE_TAG &&
                    docker stop node-app || true &&
                    docker rm node-app || true &&
                    docker run -d -p 3000:3000 --name node-app $ECR_URI:$IMAGE_TAG
                "
                '''
            }
        }
    }
}


