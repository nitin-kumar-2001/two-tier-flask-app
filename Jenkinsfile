pipeline{
    
    agent any
    
    stages{
        stage("Code Clone"){
            steps{
                git url: "https://github.com/nitin-gumber/two-tier-flask-app.git", branch: "master"
            }
        }
        stage("Build Container Image"){
            steps{
                sh "docker build -t two-tier-flask-app ."
            }
        }
        stage("Test"){
            steps{
                echo "Developer/Tester tests likh ke dega ..."
            }
        }
        stage("Push to Docker Hub"){
            steps{
                withCredentials([usernamePassword(credentialsId:"DockerHubCreds",
                    passwordVariable: "dockerHubPass",
                    usernameVariable: "dockerHubUser"
                )]){
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag two-tier-flask-app ${env.dockerHubUser}/two-tier-flask-app"
                    sh "docker push ${env.dockerHubUser}/two-tier-flask-app:latest"
                }
            }
        }
        stage("Craete and Deploy Container"){
            steps{
                sh "docker compose up -d"
            }
        }
    }
}
