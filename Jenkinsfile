pipeline {
    agent none 

    environment {
        DOCKER_HUB_USER = "nitingumberdev" 
        DOCKER_CREDS_ID = "DockerHubCreds"
        IMAGE_NAME = "two-tier-flask-app"
    }

    stages {
        stage('Build & Deploy to DEV') {
            when { branch 'test' }
            agent { label 'dev-node' } 
            steps {
                echo "Running on DEV Agent..."
                checkout scm
                sh "docker build -t ${IMAGE_NAME}:dev ."
                sh "docker rm -f flask-dev-app || true"
                sh "docker run -d -p 5000:5000 --name flask-dev-app ${IMAGE_NAME}:dev"
            }
            post {
                always { cleanWs() }
            }
        }

        stage('Build, Push & Deploy to PROD') {
            when { branch 'master' }
            agent { label 'prod-node' } 
            steps {
                echo "Running on PROD Agent..."
                checkout scm
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS_ID}", 
                    passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    echo "Building Production Image..."
                    sh "docker build -t ${USER}/${IMAGE_NAME}:latest ."
                    echo "Pushing to Docker Hub..."
                    sh "echo \$PASS | docker login -u \$USER --password-stdin"
                    sh "docker push ${USER}/${IMAGE_NAME}:latest"
                }
                echo "Deploying to Production..."
                sh "docker compose down || true"
                sh "docker compose up -d"
            }
            post {
                always { cleanWs() } // Har stage ke liye alag cleanup
            }
        }
    }
}
