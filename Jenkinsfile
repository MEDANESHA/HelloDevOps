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
                git branch: 'main', credentialsId: 'gitCred', url: 'git@github.com:MEDANESHA/deploy-k8s.git' 
                sh "yq write --inplace deployment.yaml 'spec.containers(name==hello-world-1).image' mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}" 
                withCredentials([usernamePassword(credentialsId: 'gitCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh '''
                        git config user.email "you@example.com" 
                        git config user.name "Your Name" 
                        git commit -am "Update image tag to ${env.BUILD_NUMBER}"
                        git push origin main
                    '''
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