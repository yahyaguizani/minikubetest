pipeline {
    agent any

    environment {
        KUBE_CONTEXT = "minikube"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yahyaguizani/minikubetest.git'
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'kubectl config use-context $KUBE_CONTEXT'
                    sh 'kubectl apply -f heart-backend.yaml'
                    sh 'kubectl apply -f heart-frontend.yaml'
                    sh 'kubectl rollout status deployment/heart-backend'
                    sh 'kubectl rollout status deployment/heart-frontend'
                }
            }
        }

        stage('Post Deployment Checks') {
            steps {
                script {
                    sh 'kubectl get pods -o wide'
                    sh 'kubectl get svc -o wide'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}
