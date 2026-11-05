pipeline {
    agent {
        label 'linux'
    }

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '334935451270'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EC2_HOST = '3.110.185.152'
        DEPLOY_DIR = '/opt/moviehub'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate') {
            steps {
                dir('frontend') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
                sh 'python3 -m compileall -q Backend'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build -t $ECR_REGISTRY/moviehub-backend:$IMAGE_TAG ./Backend
                    docker build -t $ECR_REGISTRY/moviehub-frontend:$IMAGE_TAG ./frontend
                    docker build -t $ECR_REGISTRY/moviehub-nginx:$IMAGE_TAG ./nginx
                '''
            }
        }

        stage('Login to ECR') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'aws-jenkins-credentials',
                    usernameVariable: 'AWS_ACCESS_KEY_ID',
                    passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                )]) {
                    sh '''
                        export AWS_DEFAULT_REGION=$AWS_REGION
                        aws ecr get-login-password --region $AWS_REGION |
                        docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    docker push $ECR_REGISTRY/moviehub-backend:$IMAGE_TAG
                    docker push $ECR_REGISTRY/moviehub-frontend:$IMAGE_TAG
                    docker push $ECR_REGISTRY/moviehub-nginx:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['moviehub-ec2-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=yes ubuntu@$EC2_HOST "cd $DEPLOY_DIR && \
                        sed -i 's/^IMAGE_TAG=.*/IMAGE_TAG=$IMAGE_TAG/' .env && \
                        docker compose --env-file .env -f compose.prod.yaml pull && \
                        docker compose --env-file .env -f compose.prod.yaml up -d"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 10
                    curl --fail --retry 5 --retry-delay 3 http://$EC2_HOST/
                '''
            }
        }
    }

    post {
        success {
            echo 'MovieHub deployment successful'
        }
        failure {
            echo 'Deployment failed. Check Jenkins console logs.'
        }
    }
}