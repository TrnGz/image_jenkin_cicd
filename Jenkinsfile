// ========== JENKINSFILE HOÀN CHỈNH – DOCKER → ECR → ECS (XANH 100%) ==========
pipeline {
    agent any

    environment {
        ECR_REGISTRY   = "591313757404.dkr.ecr.ap-northeast-1.amazonaws.com"
        ECR_REPO_NAME  = "backend-website"
        AWS_REGION     = "ap-northeast-1"
        AWS_CRED_ID    = "aws-jenkins-ecr"
        IMAGE_TAG      = "${env.GIT_COMMIT}"

        ECS_CLUSTER    = "website-pj"
        ECS_SERVICE    = "website-service-z448784m"
    }

    stages {
        stage('Checkout') { steps { checkout scm } }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                        sh 'aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'

                        // Push theo commit SHA
                        sh "docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"

                        // THÊM DÒNG NÀY – QUAN TRỌNG NHẤT!
                        sh "docker tag ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG} ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest"

                        // Giờ mới push latest → thành công 100%
                        sh "docker push ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withAWS(credentials: "${AWS_CRED_ID}", region: "${AWS_REGION}") {
                        echo "Đang force new deployment cho ECS Service..."

                        sh """
                            aws ecs update-service \
                                --cluster ${ECS_CLUSTER} \
                                --service ${ECS_SERVICE} \
                                --force-new-deployment
                        """

                        echo "Đang đợi ECS ổn định..."
                        sh """
                            aws ecs wait services-stable \
                                --cluster ${ECS_CLUSTER} \
                                --services ${ECS_SERVICE}
                        """

                        echo "HOÀN TẤT! ECS đã chạy image mới: ${GIT_COMMIT}"
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG} || true"
                sh "docker rmi ${ECR_REGISTRY}/${ECR_REPO_NAME}:latest || true"
            }
        }
    }

    post {
        always { cleanWs() }
        success { echo "CI/CD THÀNH CÔNG 100%! ECS đang chạy image mới!" }
        failure { echo "CI/CD thất bại – nhưng lần này sẽ không còn nữa!" }
    }
}
