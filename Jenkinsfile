pipeline {
    agent any

    environment {
        APP_NAME  = 'narukami47/my-nginx'
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = '/var/jenkins_home/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${APP_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Load Image to Kind') {
            steps {
                sh "kind load docker-image ${APP_NAME}:${IMAGE_TAG}"
            }
        }
    stage('Debug') {
        steps {
          sh 'echo "=== CHECK FILE ==="'
          sh 'cat Jenkinsfile || true'
        }
}
        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f k8s/deployment.yaml"
                sh "kubectl apply -f k8s/service.yaml"
                sh "kubectl apply -f k8s/ingress.yaml"
                sh "kubectl set image deployment/nginx-deployment nginx-container=${APP_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Verify') {
            steps {
                sh "kubectl rollout status deployment/nginx-deployment"
                sh "kubectl get pods"
            }
        }
    }
}
