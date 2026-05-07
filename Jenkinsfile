pipeline {
    agent any

    environment {
        AWS_REGION     = "ap-south-1"
        ECR_REPO       = "504509953970.dkr.ecr.ap-south-1.amazonaws.com/ml-project-flask"
        IMAGE_NAME     = "ml-project-flask"
        CONTAINER_NAME = "ml-app"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📥 Cloning from GitHub...'
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/Summi1610/ML_project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker Image...'
                sh 'docker build -t ${IMAGE_NAME}:latest .'
            }
        }

        stage('Push to ECR') {
            steps {
                echo '📤 Pushing image to AWS ECR...'
                withCredentials([
                    string(credentialsId: 'aws-access-key-id',
                           variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-access-key',
                           variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sh '''
                        aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                        aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                        aws configure set region ap-south-1

                        aws ecr get-login-password --region ap-south-1 | \
                        docker login --username AWS \
                        --password-stdin 504509953970.dkr.ecr.ap-south-1.amazonaws.com

                        docker tag ${IMAGE_NAME}:latest ${ECR_REPO}:latest
                        docker push ${ECR_REPO}:latest
                    '''
                }
            }
        }

        stage('Stop Old Container') {
            steps {
                echo '🛑 Stopping old container...'
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                '''
            }
        }

        stage('Run New Container') {
            steps {
                echo '🚀 Starting new container...'
                sh '''
                    docker run -d \
                        -p 5000:5000 \
                        --name ${CONTAINER_NAME} \
                        --network ml_project_default \
                        -e DB_HOST=mysql-db \
                        -e DB_USER=root \
                        -e DB_PASSWORD=root \
                        -e DB_NAME=house_db \
                        ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo '✅ Verifying deployment...'
                sh 'docker ps | grep ${CONTAINER_NAME}'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed! Image pushed to ECR and app running!'
        }
        failure {
            echo '❌ Pipeline failed! Check logs above'
        }
    }
}
