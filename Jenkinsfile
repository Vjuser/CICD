pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "499290259508.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app"
        CLUSTER_NAME = "vjcluster"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t flask-devops-app .'
            }
        }

stage('Push To ECR') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds'
        ]]) {

            sh '''
            aws ecr get-login-password --region us-east-1 | \
            docker login --username AWS \
            --password-stdin 499290259508.dkr.ecr.us-east-1.amazonaws.com

            docker tag flask-devops-app:latest \
            499290259508.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app:latest

            docker push \
            499290259508.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app:latest
            '''
        }
    }
}

stage('Deploy To EKS') {
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: 'aws-creds'
        ]]) {

            sh '''
            aws eks update-kubeconfig --region us-east-1 --name vjcluster

            helm upgrade --install flask-app ./flask-chart \
            --set image.repository=499290259508.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app \
            --set image.tag=latest
            '''
        }
    }
}
