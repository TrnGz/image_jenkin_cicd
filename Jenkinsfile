// ========== JENKINSFILE HOÀN CHỈNH – DOCKER → ECR → ECS (2025) ==========
pipeline {
    agent any

    environment {
        ECR_REGISTRY   = "591313757404.dkr.ecr.ap-northeast-1.amazonaws.com"
        ECR_REPO_NAME  = "backend-website"
        AWS_REGION     = "ap-northeast-1"
        AWS_CRED_ID    = "aws-jenkins-ecr"
        IMAGE_TAG      = "${env.GIT_COMMIT}"

        // Thông tin ECS của bạn (đã đúng)
        ECS_CLUSTER    = "website-pj"
        ECS_SERVICE    = "website-service-z448784m"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                script {
                    // Cách chạy ngon 100% mà bạn đang dùng (không cần plugin Amazon ECR)
                    withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                        sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                        sh "docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
                        sh "docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                        echo "Đang force new deployment cho ECS Service: ${ECS_SERVICE}..."

                        sh """
                            aws ecs update-service \
                                --cluster ${ECS_CLUSTER} \
                                --service ${ECS_SERVICE} \
                                --force-new-deployment
                        """

                        echo "Đang đợi ECS Service ổn định (tối đa ~10 phút)..."
                        sh """
                            aws ecs wait services-stable \
                                --cluster ${ECS_CLUSTER} \
                                --services ${ECS_SERVICE}
                        """

                        echo "HOÀN TẤT! ECS đã chạy image mới:"
                        echo "${ECR_REGISTRY}/${ECR_REPO_NAME}:${GIT_COMMIT}"
                    }
                }
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG} || true"
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest || true"
            }
        }
    }

    post {
        always {
            cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenSuccess: true)
        }
        success {
            echo "CI/CD THÀNH CÔNG 100%!"
            echo "Image mới: ${ECR_REGISTRY}/${ECR_REPO_NAME}:${GIT_COMMIT}"
            echo "ECS Service ${ECS_SERVICE} đã được update thành công!"
        }
        failure {
            echo "CI/CD THẤT BẠI – kiểm tra log ngay!"
        }
    }
}
// =====================================================================
