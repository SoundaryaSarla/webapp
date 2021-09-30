pipeline {
    agent any

    stages {
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
                sh 'docker image build -t webapp:1.0 .'
            }
        }
    }
}
