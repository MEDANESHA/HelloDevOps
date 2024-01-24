pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * *') // Check every 5 minutes
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Getting source code"
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    docker.buld("mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Azure login and push the Docker image to ACR hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azureCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh 'docker login -u ${username} -p ${password} mycontainerregistryteldahtest.azurecr.io'
                    sh "docker image push mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}"
                }
            }
        }

        // ... (other stages)

        stage('Cleanup') {
            steps {
                sh "docker rmi mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}"
                sh 'docker logout'
            }
        }
    }

    post {
        failure {
            emailext subject: 'Build Failure: ${currentBuild.fullDisplayName}',
                      body: 'The pipeline failed to build. Please check the Jenkins console output for more details.',
                      recipientProviders: [developers()],
                      to: 'mouhamedanes.h@gmail.com'
        }
    }
}
