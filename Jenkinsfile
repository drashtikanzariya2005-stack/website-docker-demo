pipeline {  
    agent any  
  
    environment {  
        AWS_REGION = 'ap-south-1'  
        ECR_REPO = 'website-docker-demo'  
        AWS_ACCOUNT_ID = '576557632072'  
        IMAGE_TAG = "${env.BUILD_NUMBER}"  
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"  
        LATEST_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"  
        DEPLOY_SERVER = '65.0.29.9'  
    }  
  
    stages {  
        stage('Checkout') {  
            steps {  
                git branch: 'main', url: 'https://github.com/drashtikanzariya2005-stack/website-docker-demo.git'  
            }  
        }  
  
        stage('Build Docker Image') {  
            steps {  
                sh 'docker build -t website-docker-demo .'  
            }  
        }  
  
        stage('Tag Docker Image') {  
            steps {  
                sh '''  
                    docker tag website-docker-demo:latest $IMAGE_URI  
                    docker tag website-docker-demo:latest $LATEST_URI  
                '''  
            }  
        }  
  
        stage('Login to ECR') {
    steps {
        sh '''
        aws ecr get-login-password --region ap-south-1 | \
        docker login --username AWS --password-stdin 576557632072.dkr.ecr.ap-south-1.amazonaws.com
        '''
    }
}
  
        stage('Push Image to ECR') {  
            steps {  
                sh '''  
                    docker push $IMAGE_URI  
                    docker push $LATEST_URI  
                '''  
            }  
        }  
  
       stage('Deploy to EC2') {
    steps {
        sshagent(['ubuntu']) {
            sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@65.0.29.9 "
            aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 576557632072.dkr.ecr.ap-south-1.amazonaws.com &&
            docker pull 576557632072.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest &&
            docker stop website-demo || true &&
            docker rm website-demo || true &&
            docker run -d --name website-demo -p 80:80 576557632072.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest
            "
            '''
        }
    }
}
    }  
  
    post {  
        success {  
            echo 'Website deployed successfully'  
        }  
        failure {  
            echo 'Pipeline failed'  
        }  
    }  
}
