pipeline {
    agent any
    tools {
        jdk 'Jdk-17'
        maven 'Maven'
    }
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '148761684695.dkr.ecr.ap-south-1.amazonaws.com/spring-petclinic'
        IMAGE_TAG = '1.0'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/venkeyboda07/Jenkins-ECR-K8S-cicd-pipeline.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar-credentials', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=spc \
                        -Dsonar.host.url=https://13.201.128.242:9000 \
                        -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker image build -t spc:$IMAGE_TAG ."
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh "aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO"
            }
        }

        stage('Tag and Push to ECR') {
            steps {
                sh "docker image tag spc:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG"
                sh "docker image push $ECR_REPO:$IMAGE_TAG"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    dir('Manifest') {  // Changed directory name to "Manifest"
                        withKubeConfig(
                            caCertificate: '', 
                            clusterName: '', 
                            contextName: '', 
                            credentialsId: 'kube-credentials', // Changed credential ID
                            namespace: '', 
                            restrictKubeConfigAccess: false, 
                            serverUrl: ''
                        ) {
                            sh 'kubectl apply -f deployment.yaml'
                            sh 'kubectl apply -f service.yaml'
                        }
                    }
                }
            }
        }
    } 
} 


