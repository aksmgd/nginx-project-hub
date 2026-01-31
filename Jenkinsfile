pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nginx-app:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                // Use your actual GitHub repo URL and credentialsId if private
                git(
                    url: 'https://github.com/aksmgd/nginx-project.git',
                    branch: 'main',
                    credentialsId: '39802547-1f4d-4d7f-9f12-a8a169ad56af'   // replace with your Jenkins credentials ID
                )
            }
        }

        stage('Build Docker Image in Minikube') {
            steps {
                // Build inside Minikube’s Docker environment
                sh "eval \$(minikube docker-env) && docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Deploy with Helm') {
            steps {
                 sh ''' 
                    helm upgrade --install nginx-app ./nginx-chart \
                         --set image.repository=nginx-app \
                         --set image.tag=latest \
                         --set image.pullPolicy=IfNotPresent 
                 '''// Deploy using Helm chart
                
            }
        }

        stage('Expose Service URL') {
            steps {
                // Print the service URL
                sh "minikube service nginx-service --url"
            }
        }
    }
}

