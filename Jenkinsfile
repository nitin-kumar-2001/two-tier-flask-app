pipeline {
    agent none // Hum global agent nahi le rahe

    stages {
        stage('Build & Deploy to Test') {
            when { branch 'test' } // Sirf 'test' branch par chalega
            agent { label 'test-agent' } // Sirf Test machine par chalega
            steps {
                sh "docker build -t flask-app-test ."
                sh "docker run -d -p 5001:5000 flask-app-test"
            }
        }

        stage('Build & Deploy to Production') {
            when { branch 'master' } // Sirf 'master' branch par chalega
            agent { label 'prod-agent' } // Sirf Production machine par chalega
            steps {
                sh "docker build -t flask-app-prod ."
                sh "docker run -d -p 80:5000 flask-app-prod"
            }
        }
    }
}
