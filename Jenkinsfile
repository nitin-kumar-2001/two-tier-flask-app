pipeline {
    agent none // Global agent none, taaki stages apne labels khud choose karein

    environment {
        DOCKER_HUB_USER = "nitingumber" 
        DOCKER_CREDS_ID = "DockerHubCreds"
        IMAGE_NAME = "two-tier-flask-app"
    }

    stages {
        // --- STAGE 1: DEV ENVIRONMENT ---
        stage('Build & Deploy to DEV') {
            when { branch 'test' } // Agar branch 'test' hai (ya aapka dev branch name)
            agent { label 'dev-node' } // Sirf dev-node machine par chalega
            steps {
                echo "Running on DEV Agent..."
                checkout scm // Sahi URL aur branch apne aap utha lega
                
                sh "docker build -t ${IMAGE_NAME}:dev ."
                
                echo "Cleaning old containers on Dev..."
                sh "docker rm -f flask-dev-app || true"
                
                echo "Starting Dev Container..."
                sh "docker run -d -p 5000:5000 --name flask-dev-app ${IMAGE_NAME}:dev"
            }
        }

        // --- STAGE 2: PRODUCTION ENVIRONMENT ---
        stage('Build, Push & Deploy to PROD') {
            when { branch 'master' } // Sirf master branch ke liye
            agent { label 'prod-node' } // Sirf prod-node machine par chalega
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
                // Yahan aap Docker Compose use kar sakte hain
                sh "docker compose down || true"
                sh "docker compose up -d"
            }
        }
    }

    post {
        always {
            cleanWs() // Workspace cleanup
            echo "Pipeline finished on ${env.BRANCH_NAME} branch"
        }
    }
}
