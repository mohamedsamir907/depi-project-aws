pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = 'mohamed907/depi-flask-app'  // Your Docker Hub repository
        GIT_REPO_URL = 'https://github.com/mohamedsamir907/depi-project-aws.git'  // Your public GitHub repository URL
        KUBECONFIG_PATH = '/root/.kube/config'  // Path to kubeconfig on Jenkins server
        K8S_DEPLOY_DIR = 'k8s/'  // Directory containing Kubernetes YAML files
    }
    stages {
        stage('Clone Repository') {
            steps {
                // Clone the Git repository
                git branch: 'main', url: 'https://github.com/mohamedsamir907/depi-project-aws.git'
            }
        }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${env.GIT_REPO_URL}"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using the Dockerfile inside the /app directory
                    dockerImage = docker.build("${env.DOCKER_HUB_REPO}:${env.BUILD_NUMBER}", "app/")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    // Push the built Docker image to Docker Hub
                    dockerImage.push()
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update the Kubernetes deployment YAML with the new image tag
                    sh """
                    sed -i 's|image: ${DOCKER_HUB_REPO}:.*|image: ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}|' ${K8S_DEPLOY_DIR}/deployment.yaml
                    """
                    // Apply the namespace, deployment, and service YAML files to the EKS cluster
                    sh """
                    kubectl --kubeconfig ${env.KUBECONFIG_PATH} apply -f ${K8S_DEPLOY_DIR}/namespace.yaml
                    kubectl --kubeconfig ${env.KUBECONFIG_PATH} apply -f ${K8S_DEPLOY_DIR}/deployment.yaml
                    kubectl --kubeconfig ${env.KUBECONFIG_PATH} apply -f ${K8S_DEPLOY_DIR}/service.yaml
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after build
        }
    }
}
