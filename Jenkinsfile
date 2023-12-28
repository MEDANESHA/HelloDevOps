pipeline {
    agent any

    environment {
        // Add the azureCred variable as authentication credentials
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
                    // Retrieve username and password from the credentials object
                    def azureCredentials = credentials(AZURE_CREDENTIALS)
                    def username = azureCredentials.username
                    def password = azureCredentials.password

                    // Authenticate with Azure Container Registry
                    docker.withRegistry('https://mycontainerregistryteldahtest.azurecr.io', username, password) {
                        // Build and push your Docker image
                        docker.build("helloworld:${env.BUILD_NUMBER}")
                        docker.withRegistry([credentialsId: AZURE_CREDENTIALS, url: 'https://mycontainerregistryteldahtest.azurecr.io']) {
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
