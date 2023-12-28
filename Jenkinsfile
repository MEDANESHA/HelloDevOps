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
                    // Retrieve the credentials object
                    def azureCredentials = credentials(AZURE_CREDENTIALS)

                    // Check if the credentials object is of type UsernamePasswordCredentials
                    if (azureCredentials instanceof org.jenkinsci.plugins.plaincredentials.StringCredentials) {
                        // Use the secret property to get the password
                        def password = azureCredentials.secret

                        // Authenticate with Azure Container Registry
                        docker.withRegistry('https://mycontainerregistryteldahtest.azurecr.io', 'username', password) {
                            // Build and push your Docker image
                            docker.build("helloworld:${env.BUILD_NUMBER}")
                            docker.withRegistry([credentialsId: AZURE_CREDENTIALS, url: 'https://mycontainerregistryteldahtest.azurecr.io']) {
                                docker.image("helloworld:${env.BUILD_NUMBER}").push()
                            }
                        }
                    } else {
                        error "Unsupported credentials type"
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
