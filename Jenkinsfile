pipeline {
    agent any

    parameters {
        choice(
            name: 'BACKEND_COUNT', 
            choices: ['1', '2'], 
            description: 'Number of backend containers to deploy'
        )
    }

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
                echo "Creating Docker network..."
                docker network create app-network || true
                
                echo "Removing old backend containers..."
                docker rm -f backend1 backend2 || true

                if [ "$BACKEND_COUNT" = "1" ]; then
                    echo "Running 1 backend container..."
                    docker run -d --name backend1 --network app-network backend-app
                else
                    echo "Running 2 backend containers..."
                    docker run -d --name backend1 --network app-network backend-app
                    docker run -d --name backend2 --network app-network backend-app
                fi
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Removing old NGINX container..."
                docker rm -f nginx-lb || true

                echo "Starting NGINX container..."
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx nginx -g "daemon off:"

                echo "Copying NGINX configuration..."
                docker cp CC_LAB-6/nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully! NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
