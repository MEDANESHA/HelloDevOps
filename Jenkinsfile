pipeline {
    agent any

    environment {
        // Use the azureCred variable directly as authentication credentials
        AZURE_CREDENTIALS = credentials('azureCred')
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

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Retrieve the credentials object
                    def azureCredentials = credentials('azureCred')

                    // Extract username and password
                    def username = azureCredentials.getUsername()
                    def password = azureCredentials.getPassword()

                    // Authenticate with Azure Container Registry
                    docker.withRegistry('https://mycontainerregistryteldahtest.azurecr.io', username, password) {
                        // Build and push your Docker image
                        docker.build("helloworld:${env.BUILD_NUMBER}")
                        docker.withRegistry([credentialsId: 'azureCred', url: 'https://mycontainerregistryteldahtest.azurecr.io']) {
                            docker.image("helloworld:${env.BUILD_NUMBER}").push()
                        }
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker rmi mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}'
                sh 'docker logout'
            }
        }
    }
}
