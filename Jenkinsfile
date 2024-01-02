pipeline {
    agent any

    environment {
        // Use the azureCred variable directly as authentication credentials
        AZURE_CREDENTIALS = credentials('azureCred')
        GIT_CREDENTIALS = credentials('gitCred') // Add your Git credentials here
    }

    triggers {
        pollSCM('*/5 * * * *') // Check every 5 minutes
    }

    stages {
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
                
        
                script {
                    // Start SSH agent and load credentials
                    sshagent(credentials: [jenkins-ssh-key]) {
                        // Set the remote URL to use SSH
                        sh 'git remote set-url origin git@github.com:MEDANESHA/deploy-k8s.git'
                        sh """
                        awk '/image: mycontainerregistryteldahtest.azurecr.io\\/helloworld:/ {sub(/mycontainerregistryteldahtest.azurecr.io\\/helloworld:[0-9]+/, "mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}");} 1' hello-world-pod.yaml > hello-world-pod-updated.yaml
                        mv hello-world-pod-updated.yaml hello-world-pod.yaml
                        """
                        // Print the content after modification
                        sh 'cat hello-world-pod.yaml'

                        // Stage the changes for commit
                        sh 'git add hello-world-pod.yaml'
                        sh 'git config user.email "mouhamed195h@gmail.com"'
                        sh 'git config user.name "medanes"'
                        sh 'git commit -am "Update image tag"'
                        sh 'git push origin main'
                    }
                    
                }
                
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