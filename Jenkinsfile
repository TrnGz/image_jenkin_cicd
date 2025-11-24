// ========== JENKINSFILE HOÀN CHỈNH – DOCKER → ECR ==========
pipeline {
    agent any

    // ---------------- THAY 4 DÒNG NÀY ----------------
    environment {
        // 1. Registry đầy đủ (copy từ ECR → View push commands)
        ECR_REGISTRY = "591313757404.dkr.ecr.ap-northeast-1.amazonaws.com"   
        
        // 2. Tên repository bạn đã tạo trong ECR
        ECR_REPO_NAME = "backend-website"                                      
        
        // 3. Region của bạn (ví dụ: ap-northeast-1, us-east-1, ap-southeast-1…)
        AWS_REGION    = "ap-northeast-1"                                   
        
        // 4. ID credential AWS bạn đã tạo (bắt buộc đúng 100%)
        AWS_CRED_ID   = "aws-jenkins-ecr"                                  
    }
    // --------------------------------------------------

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Tag theo commit SHA (rất sạch, dễ rollback)
                    def imageTag = env.GIT_COMMIT
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPO_NAME}:${imageTag}")
                }
            }
        }

        stage('Push to Amazon ECR') {
            steps {
                script {
                    // Cách hiện đại nhất 2025 – dùng plugin Amazon ECR (không cần docker login thủ công)
                    docker.withRegistry("https://${ECR_REGISTRY}", "ecr:${AWS_REGION}:${AWS_CRED_ID}") {
                        dockerImage.push()              // push theo commit SHA
                        dockerImage.push('latest')      // push thêm tag latest cho tiện
                    }
                }
            }
        }

        stage('Cleanup Local Images') {
            steps {
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:${env.GIT_COMMIT} || true"
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest || true"
            }
        }
    }
stage('Deploy to ECS') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                        echo "Đang force new deployment cho ECS Service..."
                        
                        // Force ECS Service chạy lại với image mới nhất
                        sh """
                        aws ecs update-service \
                            --cluster website-pj \
                            --service website-service-z448784m \
                            --force-new-deployment \
                            --region ${AWS_REGION}
                        """
                        
                        // (Tùy chọn) Đợi ECS ổn định – cực kỳ hay
                        sh """
                        aws ecs wait services-stable \
                            --cluster website-pj \
                            --services website-service-z448784m \
                            --region ${AWS_REGION}
                        """
                        
                        echo "Deploy thành công! ECS đã chạy image mới:"
                        echo "${ECR_REGISTRY}/${ECR_REPO_NAME}:${GIT_COMMIT}"
                    }
                }
            }
        }
    post {
        always {
            cleanWs()   // dọn workspace, tiết kiệm ổ cứng
        }
        success {
            echo "✅ Đã push image thành công lên ECR!"
            echo "Image: ${ECR_REGISTRY}/${ECR_REPO_NAME}:${env.GIT_COMMIT}"
            echo "Latest: ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest"
        }
        failure {
            echo "❌ Build thất bại – kiểm tra log ngay!"
        }
    }
}
// =========================================================
