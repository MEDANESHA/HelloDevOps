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
                    docker.build("mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}")
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
         stage('Update Helm Values File') {
            steps {
                script {
                    // Variables
                    def helmRepoUrl = 'git@github.com:MEDANESHA/deploy-k8s.git' // Replace with your repo URL
                    def branchName = 'main' // Replace with target branch
                    def helmChartPath = 'hello-world-charts/values.yaml' // Path to values.yaml

                    // Clone the Helm repo
                    sh """
                        rm -rf helm-repo
                        git clone ${helmRepoUrl} helm-repo
                    """
                    dir('helm-repo') {
                        // Checkout the desired branch
                        sh """
                            git checkout ${branchName}
                        """

                        // Update the image tag in values.yaml
                        sh """
                            sed -i 's|tag:.*|tag: "${env.BUILD_NUMBER}"|' ${helmChartPath}
                        """

                        // Commit and push the change
                        sh """
                        git config user.email "ci-bot@example.com"
                        git config user.name "CI Bot"
                        git add ${helmChartPath}
                        git commit -m "Update image tag to ${env.BUILD_NUMBER}"
                        git push origin ${branchName}
                        """
                        }
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
                      to: 'mouhamed195h@gmail.com'
        }
    }
}
