pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '499290259508.dkr.ecr.us-east-1.amazonaws.com/flask-devops-app'
        CLUSTER_NAME = 'vjcluster'
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
                    aws ecr get-login-password --region $AWS_REGION | \
                    docker login --username AWS --password-stdin 499290259508.dkr.ecr.us-east-1.amazonaws.com

                    docker tag flask-devops-app:latest $ECR_REPO:latest

                    docker push $ECR_REPO:latest
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
                    aws eks update-kubeconfig --region $AWS_REGION --name $CLUSTER_NAME

                    helm upgrade --install flask-app ./flask-chart
                    '''
                }
            }
        }
    }
}
