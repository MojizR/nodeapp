pipeline {
    agent any
    environment{
        registry = "mojizrahman/web-dev"
        registryCredential = "docker-cred"
        dockerImage = ''
    }
    stages {
        stage('Building Dockerimage') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage = docker.build( registry + ":app-$BUILD_NUMBER")
                    }
                }
            }
        }
        stage('Pushing Dockerimage Into Dockerhub') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
 //       stage('Remove Unused Dockerimage') {
 //          steps{
 //              sh "docker rmi $registry:app-$BUILD_NUMBER"
 //         }
 //     }
        stage('Secrets Copy') {
            steps{
                    withCredentials([file(credentialsId: 'aws-docker-ec2-cred', variable: 'serverkey')]) {
                        sh "cp \$serverkey web1.pem"
                }
            }
        }
        stage('docker Deploy') {
            steps{
                sh 'chmod 400 web1.pem'
                sh 'ssh -o StrictHostKeyChecking=no -i web1.pem ec2-user@3.142.164.107 sudo docker run --name web-dev-app --restart=always -p 3000:3000 -d $registry:app-$BUILD_NUMBER'
                sh 'ssh -o StrictHostKeyChecking=no -i web1.pem ec2-user@3.142.164.107 sudo docker system prune -f'
            }
        }
        stage('Workspace Cleanup') {
            steps{
                cleanWs()
            }
        }
    }
}
