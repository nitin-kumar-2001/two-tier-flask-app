pipeline {
    agent none 
    environment {
        IMAGE_NAME = "nitingumberdev/two-tier-flask-app"
        DOCKER_CREDS = "DockerHubCreds"
    }

    stages {
        // --- DEV FLOW ---
        stage('Build & Deploy to Dev') {
            when { branch 'dev' }
            agent { label 'dev-node' } // Pura kaam isi machine par hoga
            steps {
                checkout scm // Yahan 'any' ki jagah isi machine par clone karo
                sh "docker build -t ${IMAGE_NAME}:dev ."
                
                // Docker Compose use karna best hai
                sh "docker compose down || true" 
                sh "docker compose up -d"
                echo "Dev site ready on Dev Agent IP!"
            }
        }

        // --- PROD FLOW (Centralized Image) ---
        stage('Production Pipeline') {
            when { branch 'master' }
            // Yahan hum pehle image banakar push karenge (kisi bhi machine par)
            // Phir Prod Agent par pull karke deploy karenge
            stages {
                stage('Build & Push') {
                    agent any
                    steps {
                        checkout scm
                        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDS}", 
                            passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                            sh "docker build -t ${IMAGE_NAME}:latest ."
                            sh "echo $PASS | docker login -u $USER --password-stdin"
                            sh "docker push ${IMAGE_NAME}:latest"
                        }
                    }
                }
                stage('Deploy to Prod') {
                    agent { label 'prod-node' }
                    steps {
                        // Prod Agent par Docker Hub se image apne aap pull ho jayegi via Compose
                        sh "docker compose pull"
                        sh "docker compose up -d"
                    }
                }
            }
        }
    }
}
