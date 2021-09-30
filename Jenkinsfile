pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="869250677914"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="webapp"
        DOCKER_HUB_ID="raajesh404"
        IMAGE_TAG="1.0"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
    stages {
        stage('Logging into AWS ECR') {
            steps {
                sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
            }
        }
        stage('clone the repo from github') {
            steps {
                git 'https://github.com/RajeshAudhurthi/webapp.git'
            }
        }
        stage('maven build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('archive the artifacts:war') {
            steps {
                archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
                sh 'mv target/*war .'
            }
        }
        stage('docker image build') {
            steps {
                sh 'docker image build -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} .'
            }
        }
    }
}
