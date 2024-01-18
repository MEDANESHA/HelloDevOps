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
                    
                    
                   script {
                    sh """
                        awk '/repository: mycontainerregistryteldahtest.azurecr.io\\/helloworld:/ {sub(/mycontainerregistryteldahtest.azurecr.io\\/helloworld:[0-9]+/, "mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}");} 1' hello-world-charts/values.yaml > hello-world-charts/values-updated.yaml
                        mv hello-world-charts/values-updated.yaml hello-world-charts/values.yaml
                        """
}

                }
                     // Update Helm chart version in Chart.yaml
                    sh """
                        awk '/^version:/ {sub(/[0-9]+\\.[0-9]+\\.[0-9]+/, "${env.BUILD_NUMBER}");} 1' hello-world-charts/Chart.yaml > Chart-updated.yaml
                        mv Chart-updated.yaml hello-world-charts/Chart.yaml
                        """
                
                    // Print the content after modification
                    sh 'cat values.yaml'
                   
    
                    // Stage the changes for commit
                    sh 'git add values.yaml'
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