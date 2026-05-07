pipeline {
    agent any

    environment {
        IMAGE_NAME = "ml-project-flask"
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
            echo '✅ Pipeline completed successfully! App is running on port 5000'
        }
        failure {
            echo '❌ Pipeline failed! Check logs above'
        }
    }
}
