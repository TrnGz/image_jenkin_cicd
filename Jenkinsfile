pipeline {
    agent any

    environment {
        AWS_REGION    = "ap-northeast-1"                  // Thay region của bạn
        ACCOUNT_ID    = "591313757404"                    // Thay bằng AWS Account ID của bạn
        ECR_REPO      = "backend-website"                  // Tên ECR repository
        IMAGE_TAG     = "${BUILD_NUMBER}"
        ECS_CLUSTER   = "website-pj"                  // Tên ECS Cluster
        ECS_SERVICE   = "web-01-service-kqv6l5zg"                  // Tên ECS Service
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh """
                aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --force-new-deployment
                """
            }
        }
    }

    post {
        always {
            sh "docker rmi ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG} || true"
        }
        success {
            echo "CI/CD THÀNH CÔNG! Ứng dụng đã được deploy phiên bản ${IMAGE_TAG}"
        }
    }
}
