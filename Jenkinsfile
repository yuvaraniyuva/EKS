pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-2'
        ECR_REPO = '533267389601.dkr.ecr.us-east-1.amazonaws.com/nginxapp'
        EKS_CLUSTER = 'my-cluster'
 //     KUBECONFIG = credentials('kubeconfig-credentials-id')
        registryCredential = 'aws'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yuvaraniyuva/EKS.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REPO}:latest")
                }
            }
        }
        stage('Login to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws']]) {
                    sh '''
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 533267389601.dkr.ecr.us-east-1.amazonaws.com
                    '''
                }
            }
        }
        // Other stages...
        stage('Push to ECR') {
            steps {
                script {
                    docker.withRegistry("533267389601.dkr.ecr.us-east-1.amazonaws.com", registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    sh """
                    aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER}
                    kubectl apply -f k8s/deployment.yml
                    """
                }
            }
        }
    }
}
