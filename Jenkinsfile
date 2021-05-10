pipeline {
    agent any
    environment {
        //be sure to replace "willbla" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "jenkinsmeetup/train-app"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            
            steps {
                input 'Deploy to Production?'
                milestone(1)
                 withCredentials([usernamePassword(credentialsId: 'github_cred', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                   sh("git clone https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Rajdattpawar/argocd-demo-deploy.git")
                   sh "cd argocd-demo-deploy/prod && /var/lib/jenkins/kustomize edit set image jenkinsmeetup/train-app:${env.BUILD_NUMBER}"
                   sh "git commit -am 'Publish new version' && git push  https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/Rajdattpawar/argocd-demo-deploy.git || echo 'no changes'"
                  }
                //implement Kubernetes deployment here
                
            }
        }
    }
}
