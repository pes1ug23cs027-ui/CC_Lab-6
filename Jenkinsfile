pipeline {
    agent any

    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend Docker image..."
                docker rmi -f backend-app || true
                docker build -t backend-app CC_LAB-6/backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create network if not exists
                docker network inspect app-network >/dev/null 2>&1 || docker network create app-network

                # Remove existing containers if any
                docker rm -f backend1 backend2 >/dev/null 2>&1 || true

                # Run backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Remove existing nginx container
                docker rm -f nginx-lb >/dev/null 2>&1 || true

                # Run nginx container
                docker run -d --name nginx-lb --network app-network -p 80:80 nginx:latest

                # Copy config and reload nginx
                docker cp CC_LAB-6/nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                docker exec nginx-lb nginx -g 'daemon off;' &
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
