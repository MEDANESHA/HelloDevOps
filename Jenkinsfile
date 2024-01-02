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
            sh """sed -i 's|spec.containers(name==hello-world-1).image.*|spec.containers(name==hello-world-1).image mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}|' hello-world-pod.yaml"""
        
            // Print the content after modification
            sh 'cat hello-world-pod.yaml'
        
            // Stage the changes for commit
            sh 'git add hello-world-pod.yaml'
        
            withCredentials([usernamePassword(credentialsId: 'gitCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                sh '''
                    git config user.email "you@example.com" 
                    git config user.name "Your Name" 
                    git commit -am "Update image tag
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