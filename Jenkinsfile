pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * *') // Check every 5 minutes
    }

    stages {
        stage('Checkout') {
            steps {
                echo "getting source code"
                checkout scm 
                }
            }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.build("mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('azure login and push the docker image to acr hub'){
            steps{
                withCredentials ([usernamePassword(credentialsId: 'azureCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh 'docker login -u ${username} -p ${password} mycontainerregistryteldahtest.azurecr.io'
                sh "docker image push mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}"
        
            }
        }
        }
        stage('Update K8s Manifest') {
            steps {
                git branch: 'main', credentialsId: 'jenkins-ssh-key', url: 'git@github.com:MEDANESHA/deploy-k8s.git'
        
                script {
                    
                    
                    sh """
                        awk '/image: mycontainerregistryteldahtest.azurecr.io\\/helloworld:/ {sub(/mycontainerregistryteldahtest.azurecr.io\\/helloworld:[0-9]+/, "mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}");} 1' hello-world-pod.yaml > hello-world-pod-updated.yaml
                        mv hello-world-pod-updated.yaml hello-world-pod.yaml
                        awk '/image: mycontainerregistryteldahtest.azurecr.io\\/helloworld:/ {sub(/mycontainerregistryteldahtest.azurecr.io\\/helloworld:[0-9]+/, "mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}");} 1' hello-world-deploy.yaml > hello-world-deploy-updated.yaml
                        mv hello-world-deploy-updated.yaml hello-world-deploy.yaml
                    """
                }
                
                    // Print the content after modification
                    sh 'cat hello-world-pod.yaml'
                    sh 'cat hello-world-deploy.yaml'
    
                    // Stage the changes for commit
                    sh 'git add hello-world-pod.yaml'
                    sh """
                       
                        git commit -am "Update image tag to ${env.BUILD_NUMBER}"
                        
                        git push  origin main

                    """
        
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}"
                sh 'docker logout'
            }
        }
    }
}