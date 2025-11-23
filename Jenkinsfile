pipeline {
    agent any

    environment {
        AWS_REGION    = "ap-northeast-1"
        ACCOUNT_ID    = "591313757404"                   // ← sửa đúng Account ID của bạn
        ECR_REPO      = "backend-website"                // ← tên ECR repo
        IMAGE_TAG     = "${BUILD_NUMBER}"
        ECS_CLUSTER   = "website-pj"
        ECS_SERVICE   = "web-01-service-kqv6l5zg"        // ← tên service chính xác
    }

    stages {
        // Bỏ stage Checkout Code riêng vì Pipeline script from SCM đã tự checkout rồi
        // → để lại sẽ bị checkout 2 lần (nguyên nhân log bị lặp như bạn thấy)

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS --password-stdin ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    
                    docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}
                    docker push ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest
                '''
            }
        }

        stage('Deploy to ECS') {
            steps {
                sh '''
                    aws ecs update-service \
                        --cluster ${ECS_CLUSTER} \
                        --service ${ECS_SERVICE} \
                        --force-new-deployment
                '''
            }
        }
    }

    post {
        always {
            sh '''
                docker rmi ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG} || true
                docker rmi ${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest || true
            '''
        }
        success {
            echo "CI/CD THÀNH CÔNG 100%! Phiên bản ${IMAGE_TAG} đã được deploy lên ECS"
        }
        failure {
            echo "CI/CD thất bại – kiểm tra log trên"
        }
    }
}
