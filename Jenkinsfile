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