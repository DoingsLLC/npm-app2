pipeline {
    agent any

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        ACR_NAME = "doingsacr"
        registryUrl = "doingsacr.azurecr.io"
        IMAGE_NAME = "doings/nodejs-app"
        IMAGE_TAG = "v1"
        registryCredential = "doingsacr-credential" // Make sure this references the correct credential ID
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DoingsLLC/npm-app2.git'
            }
        }

        stage('Run Sonarqube') {
            environment {
                scannerHome = tool 'ibt-sonarqube'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'ibt-sonar', installationName: 'IBT sonarqube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    def dockerImage = docker.build("${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}", '.')
                }
            }
        }

        stage('Upload Image to ACR') {
            steps {
                script {
                    docker.withRegistry("http://${ACR_NAME}.azurecr.io", registryCredential) {
                        def dockerImage = docker.image("${ACR_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}")
                        dockerImage.push() // Uncomment this line to push the image
                    }
                }
            }
        }
    }
}
