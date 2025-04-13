pipeline {
    agent any

    environment {
        IMAGE_NAME = 'rnkh41/python'
    }

    stages {
        stage('Clone Repo') {
            steps {
                git branch: 'main', url: 'https://github.com/avvaraj/python_app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${IMAGE_NAME}:${BUILD_NUMBER}"
                    dockerImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                        dockerImage.push()
                        // Also push the 'latest' tag so K8s always pulls the latest
                        dockerImage.tag('latest')
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Force Restart Deployment') {
            steps {
                echo "Triggering rolling update in Kubernetes..."
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    // Reapply deployment.yaml and force restart to pull new image
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl rollout restart deployment/python-app -n jenkins'
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs above."
        }
    }
}
