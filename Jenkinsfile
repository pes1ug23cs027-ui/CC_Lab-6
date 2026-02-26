pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "backend-app"
        NETWORK_NAME = "app-network"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                echo "Building backend Docker image..."
                sh '''
                docker rmi -f $BACKEND_IMAGE || true
                docker build -t $BACKEND_IMAGE backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                echo "Deploying backend containers..."
                sh '''
                docker network create $NETWORK_NAME || true
                docker rm -f backend1 backend2 || true
                docker run -d --name backend1 --network $NETWORK_NAME -p 8081:8080 $BACKEND_IMAGE
                docker run -d --name backend2 --network $NETWORK_NAME -p 8082:8080 $BACKEND_IMAGE
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                echo "Deploying NGINX load balancer..."
                sh '''
                docker rm -f nginx-lb || true
                docker run -d --name nginx-lb --network $NETWORK_NAME -p 80:80 nginx
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -s reload || true
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully. Backend containers and NGINX load balancer are running."
        }
        failure {
            echo "Pipeline failed. Check console logs for errors."
        }
    }
}
