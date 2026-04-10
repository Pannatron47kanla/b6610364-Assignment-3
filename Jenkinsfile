pipeline {
    agent any   // ❗ เปลี่ยนจาก label เพราะคุณยังไม่มี k8s-agent

    triggers {
        githubPush()
    }

    environment {
        APP_NAME  = 'narukami47/my-nginx'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${APP_NAME}:${IMAGE_TAG} .
                docker tag ${APP_NAME}:${IMAGE_TAG} ${APP_NAME}:latest
                """
            }
        }

        stage('Load Image to kind') {
            steps {
                sh """
                kind load docker-image ${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl apply -f k8s/ingress.yaml
                """
            }
        }

        stage('Update Image (Rolling Update)') {
            steps {
                sh """
                kubectl set image deployment/my-nginx nginx=${APP_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                sh """
                kubectl rollout status deployment/my-nginx --timeout=120s
                kubectl get pods -l app=my-nginx
                kubectl get svc my-nginx-service
                kubectl get ingress my-nginx-ingress
                """
            }
        }
    }

    post {
        success {
            echo "Deployment successful! Access at http://my-nginx.local"
        }
        failure {
            echo "Deployment failed! Check logs."
        }
    }
}
