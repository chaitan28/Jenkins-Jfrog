pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "myapp:${env.BUILD_ID}"   // Docker image with a unique tag
        ARTIFACTORY_REPO = "docker-local"        // JFrog Artifactory repository name
        ARTIFACTORY_CREDENTIALS = 'artifactory-credentials' // JFrog credentials ID
        KUBECONFIG_CREDENTIALS = 'k8s-credentials'          // Kubernetes credentials ID
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                // Build Java application and run tests
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push to JFrog Artifactory') {
            steps {
                script {
                    // Use JFrog CLI to log in and push the image
                    withCredentials([usernamePassword(credentialsId: ARTIFACTORY_CREDENTIALS, 
                                usernameVariable: 'ARTIFACTORY_USER', 
                                passwordVariable: 'ARTIFACTORY_API_KEY')]) {
                        sh 'echo $ARTIFACTORY_API_KEY | docker login -u $ARTIFACTORY_USER --password-stdin myjfrogurl/artifactory/docker-local'
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: KUBECONFIG_CREDENTIALS, variable: 'KUBECONFIG')]) {
                    // Apply Kubernetes deployment YAML
                    sh 'kubectl apply -f k8s/deployment.yaml'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                // Verify the deployment status
                sh 'kubectl rollout status deployment/myapp'
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
        always {
            cleanWs()
        }
    }
}
