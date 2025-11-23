pipeline {
    agent any
    environment {
        AWS_REGION      = "ap-northeast-1"
        ACCOUNT_ID      = "591313757404"
        ECR_REPO        = "image_jenkin_cicd"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        ECS_CLUSTER     = "shoppe-cluster"
        ECS_SERVICE     = "shoppe-service"
    }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build Image') {
            steps {
                script {
                    docker.build("${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }
        stage('Push ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest
                """
            }
        }
        stage('Deploy ECS') {
            steps {
                sh "aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment"
            }
        }
    }
}
