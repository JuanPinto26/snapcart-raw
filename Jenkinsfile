pipeline {
    agent any

    environment {
        IMAGE_NAME = 'snapcart'
        IMAGE_TAG  = 'latest'
        NAMESPACE  = 'snapcart'
        K8S_DIR    = 'k8s'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Pulling latest code from GitHub...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Test') {
            steps {
                sh """
                    docker rm -f snapcart-test 2>/dev/null || true
                    docker run -d --name snapcart-test -p 3001:3000 ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 10

                    STATUS=\$(curl -s -o /dev/null -w "%{http_code}" http://host.docker.internal:3001/api/health)

                    docker stop snapcart-test && docker rm snapcart-test

                    if [ "\$STATUS" != "200" ]; then
                        echo "Health check failed — HTTP \$STATUS"
                        exit 1
                    fi
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes (simulated)...'
                sh 'echo Deployment completed successfully'
            }
        }
    }

    post {
        always {
            sh 'docker rm -f snapcart-test 2>/dev/null || true'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check the logs above.'
        }
    }
}