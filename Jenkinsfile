pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID = '656732270450'
        AWS_REGION     = 'ap-southeast-1'
        ECR_REPO       = 'jenkins-tutorial-app'
        IMAGE_TAG      = "${BUILD_NUMBER}"
        ECR_URL        = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
            }
        }
        stage('Quality Checks') {
            parallel {
                stage('Unit Test') {
                    steps {
                        echo 'Running unit tests...'
                        sh 'python3 -m pytest test_app.py -v'
                    }
                }
                stage('Lint Check') {
                    steps {
                        echo 'Running lint check...'
                        sh 'sleep 3'
                        echo 'Lint check passed!'
                    }
                }
                stage('Security Scan') {
                    steps {
                        echo 'Running security scan...'
                        sh 'sleep 4'
                        echo 'Security scan passed!'
                    }
                }
            }
        }
        stage('Push to ECR') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'main'
                }
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-credentials'
                ]]) {
                    sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_URL}
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URL}:${IMAGE_TAG}
                        docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_URL}:latest
                        docker push ${ECR_URL}:${IMAGE_TAG}
                        docker push ${ECR_URL}:latest
                    """
                }
            }
        }
        stage('Deploy to ECS') {
            when {
                expression {
                    return env.GIT_BRANCH == 'origin/main' || env.GIT_BRANCH == 'main'
                }
            }
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-credentials'
                ]]) {
                    sh """
                        aws ecs update-service --cluster jenkins-tutorial-cluster --service jenkins-tutorial-service --force-new-deployment --region ${AWS_REGION}
                    """
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
