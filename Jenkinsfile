pipeline {
    agent any
    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '148761684695.dkr.ecr.ap-south-1.amazonaws.com/spring-petclinic:1.0'
        IMAGE_TAG = '1.0'
        SONAR_HOST_URL = 'https://13.201.128.242:9000'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/venkeyboda07/Jenkins-ECR-K8S-cicd-pipeline.git', branch: 'main'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t spc:1.0 .'
            }
        }
        
        stage('Push to ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws cred']]) {
                    sh '''
                    aws ecr get-login-password --region ap-south-1 | \
                    docker login --username AWS --password-stdin 148761684695.dkr.ecr.ap-south-1.amazonaws.com
        
                    docker tag spc:1.0 148761684695.dkr.ecr.ap-south-1.amazonaws.com/spring-petclinic:1.0
        
                    docker push 148761684695.dkr.ecr.ap-south-1.amazonaws.com/spring-petclinic:1.0
                    '''
                }
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=my-app -Dsonar.host.url=$SONAR_HOST_URL'
                }
            }
        }
    }
}