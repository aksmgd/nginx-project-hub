pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "aksmgd/nginx_dockerhub:${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git(
                    url: 'https://github.com/aksmgd/nginx-project-hub.git',
                    branch: 'main',
                    credentialsId: 'github-creds'   
                )
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE
                    docker logout
                    '''
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                helm upgrade --install nginx-release ./nginx-chart \
                     --set image.repository=aksmgd/nginx_dockerhub \
                     --set image.tag=${BUILD_NUMBER} \
                     --set image.pullPolicy=Always \
                     --wait --timeout 60s
                '''
            }
        }

        stage('Wait for Pod Ready') {
            steps {
                sh "kubectl rollout status deployment/nginx-release --timeout=60s"
            }
        }

        stage('Expose Service URL') {
            steps {
                sh "kubectl get svc nginx-service"
            }
        }
    }
}
