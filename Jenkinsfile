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
                git branch: 'main', credentialsId: 'jenkins-ssh-key', url: 'https://github.com/MEDANESHA/deploy-k8s.git'
        
                script {
                    sh """
                        awk '/image: mycontainerregistryteldahtest.azurecr.io\\/helloworld:/ {sub(/mycontainerregistryteldahtest.azurecr.io\\/helloworld:[0-9]+/, "mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}");} 1' hello-world-pod.yaml > hello-world-pod-updated.yaml
                        mv hello-world-pod-updated.yaml hello-world-pod.yaml
                    """
                }
                // Set the remote URL to use SSH
                sh 'git remote set-url origin git@github.com:MEDANESHA/deploy-k8s.git'

                // Print the content after modification
                sh 'cat hello-world-pod.yaml'
    
                // Stage the changes for commit
                sh 'git add hello-world-pod.yaml'
                sh '''
                        git config user.email "you@example.com" 
                        git config user.name "Your Name" 
                        git commit -am "Update image tag"
                        sh 'echo "Running ssh -Tv git@github.com"'
                        sh 'ssh -Tv git@github.com'

                        git push --set-upstream origin main

                    '''
        
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