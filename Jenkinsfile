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

        stage('Push to ACR') {
            steps {
                script {
                    // Authenticate with Azure Container Registry
                    azureContainerRegistry(
                        azureCredentialsId: '$AZURE_CREDENTIALS',
                        registryType: 'AzureRM'
                    )

                    // Push Docker image to ACR
                    docker.withRegistry('https://mycontainerregistryteldahtest.azurecr.io', '$AZURE_CREDENTIALS') {
                        docker.image("mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up - remove the Docker image from the local Docker daemon
            script {
                docker.image("mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}").remove()
            }
        }
    }
}
