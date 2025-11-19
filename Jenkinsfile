pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS= credentials('dockerhub')
        BACKEND_IMAGE = "yahyaguizani/minikubeimages:heart-backend"
        FRONTEND_IMAGE = "yahyaguizani/minikubeimages:heart-frontend"
        KUBE_CONTEXT = "minikube"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/yahyaguizani/minikubetest.git'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        script {
                            sh 'docker build -t $BACKEND_IMAGE ./backend'
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        script {
                            sh 'docker build -t $FRONTEND_IMAGE ./frontend'
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --pasword-stdin'
                    sh 'docker push $BACKEND_IMAGE'
                    sh 'docker push $FRONTEND_IMAGE'
                }
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
            // ممكن تضيف Slack notification أو Email هنا
        }
    }
}
