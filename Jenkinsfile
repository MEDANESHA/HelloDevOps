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
        stage('Azure login and push the Docker image to ACR hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'azureCred', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh 'docker login -u ${username} -p ${password} mycontainerregistryteldahtest.azurecr.io'
                    sh "docker image push mycontainerregistryteldahtest.azurecr.io/helloworld:${env.BUILD_NUMBER}"
                }
            }
        }
        stage('Update K8s Manifest') {
            steps {
                git branch: 'main', credentialsId: 'jenkins-ssh-key', url: 'git@github.com:MEDANESHA/deploy-k8s.git'

                script {
                    def newTag = "${env.BUILD_NUMBER}"

                    // Update the tag attribute in values.yaml
                    sh """
                        sed -i.bak 's/tag: "[0-9]\\+"/tag: "${newTag}"/' hello-world-charts/values.yaml
                    """
                }

                // Update Helm chart version in Chart.yaml
                sh """
                    awk '/^version:/ {sub(/[0-9]+\\.[0-9]+\\.[0-9]+/, "${env.BUILD_NUMBER}");} 1' hello-world-charts/Chart.yaml > Chart-updated.yaml
                    mv Chart-updated.yaml hello-world-charts/Chart.yaml
                """

                // Print the content after modification
                sh 'cat hello-world-charts/blues.yaml'

                // Stage the changes for commit
                sh 'git add hello-world-charts/bues.yaml'
                sh """
                    git commit -am "Update image tag to ${env.BUILD_NUMBER}"
                    git push origin main
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

    post {
        failure {
            script {
                emailext subject: 'Build Failed: ${currentBuild.fullDisplayName}',
                          body: 'The build failed. Please check the Jenkins console output for more details.',
                          to: 'mouhamedanes.h@gmail.com',
                          attachLog: true
            }
        }
    }
}
