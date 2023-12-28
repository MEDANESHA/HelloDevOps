pipeline {
    agent any
    environment {
        // Ajouter la variable azureCred comme variables d'authentification
        AZURE_CREDENTIALS = credentials('azureCred')
                }
    triggers {
        pollSCM('*/5 * * * *') // Vérifier toutes les 5 minutes
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
         stage('Build and Push Docker Image') {
            steps {
                script {
                    // Authenticate with Azure Container Registry
                    docker.withRegistry('https://mycontainerregistryteldahtest.azurecr.io', '$AZURE_CREDENTIALS') {
                        // Build and push your Docker image
                        docker.build("helloworld:${env.BUILD_NUMBER}")
                        docker.withRegistry([credentialsId: '$AZURE_CREDENTIALS', url: 'https://mycontainerregistryteldahtest.azurecr.io']) {
                            docker.image("helloworld:${env.BUILD_NUMBER}").push()
                        }
                    }
                }
            }
        }

        
        stage('Cleanup'){
            steps {
                sh 'docker rmi mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}'
                sh 'docker logout'
                }
                }
    }
}

   
