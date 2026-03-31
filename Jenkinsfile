pipeline {
    agent any

    environment {
        IMAGE_NAME = 'snapcart'
        IMAGE_TAG  = 'latest'
        NAMESPACE  = 'snapcart'
        K8S_DIR    = 'k8s'
    }

    stages {
        // all stages go here

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

        stage('Deploy to Kubernetes') {
            steps {
                // 1. Define it as TEMP_KUBECONFIG
                withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'TEMP_KUBECONFIG')]) {
                    sh '''
                # 2. Download & Prep kubectl
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x ./kubectl

                # 3. Use the MATCHING variable name ($TEMP_KUBECONFIG)
                # This now correctly reads the secret file and fixes the IP
                sed "s/127.0.0.1/kubernetes.docker.internal/g" $TEMP_KUBECONFIG > ./kubeconfig.modified

                # 4. Point kubectl to the NEW modified file
                export KUBECONFIG=./kubeconfig.modified

                # 5. Apply manifests
                ./kubectl apply -f k8s/namespace.yaml --validate=false
                ./kubectl apply -f k8s/deployment.yaml --validate=false
                ./kubectl apply -f k8s/service.yaml --validate=false

                # Note: If ${NAMESPACE} is a Jenkins variable, ensure you are using double quotes for the sh block
                # OR use the namespace name directly.
                ./kubectl rollout status deployment/snapcart-deployment -n snapcart --timeout=120s
            '''
                }
            }
        }
    }
}
