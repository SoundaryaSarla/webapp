pipeline {
    agent any
    environment {
        AWS_ACCOUNT_ID="869250677914"
        AWS_DEFAULT_REGION="us-east-1"
        IMAGE_REPO_NAME="web-app"
        DOCKER_HUB_ID="raajesh404"
        IMAGE_TAG="1.0"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
   
    stages {

        
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }
        stage('Clone the repo') {
            steps {
                echo 'clone the repo'
                script {
                    sh "rm -rf game-of-life"
                }
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/RajeshAudhurthi/game-of-life.git']]])
                
            }
        }
        stage('build from maven') {
            steps {
                echo 'building war file using maven'
                script {
                    sh "mvn package"
                }
                
            }
        }
        stage('Archive the artifacts') {
            steps {
                echo 'Archive the artifacts'
                archiveArtifacts artifacts: 'gameoflife-web/target/*.war', followSymlinks: false
                script {
                    sh "cp gameoflife-web/target/*.war ."
                }
            }
        }
  
        // Building Docker images
        stage('Building image') {
          steps{
            script {
              dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
          }
        }
   
        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
            steps{  
                script {
                        sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
                }
                }
            }
        stage('Pushing to Docker-Hub') {
            steps{  
                script {
                        withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerhubPWD')]) {
                            sh "docker login -u ${DOCKER_HUB_ID} -p ${dockerhubPWD}"
                    }
                        
                        sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${DOCKER_HUB_ID}/${IMAGE_REPO_NAME}:$IMAGE_TAG"
                        sh "docker push ${DOCKER_HUB_ID}/${IMAGE_REPO_NAME}:$IMAGE_TAG"
                }
                }
        }
        stage('docker deploy') {
            steps {
                ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: '-e DOCKER_TAG="${IMAGE_TAG}"', installation: 'ansible', inventory: 'docker-inventory.inv', playbook: 'docker-playbook.yml'   
            }
        }
        stage('k8s deploy') {
            steps {
                sshagent(['kubeadm']) {
                    // some block
                    sh "scp -o StrictHostKeyChecking=no deployment-gol.yml service-gol-np.yml ubuntu@34.234.90.127:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@34.234.90.127 sudo kubectl apply -f ."
                        }
                        catch(error){
                            sh "ssh ubuntu@34.234.90.127 sudo kubectl create -f ."
                        }
                    }
                }
            }
        }

   
    }
}