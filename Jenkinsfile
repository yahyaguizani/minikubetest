pipeline {
    agent any

    parameters {
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Select environment')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
    }

    environment {
        DOCKER_USER = credentials('dockerhub-username')
        DOCKER_PASS = credentials('dockerhub-password')
        SLACK_CHANNEL = "#ci-cd-notifications"
        KUBE_CONTEXT_DEV = "minikube-dev"
        KUBE_CONTEXT_STAGING = "minikube-staging"
        KUBE_CONTEXT_PROD = "minikube-prod"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${params.BRANCH}", url: 'https://github.com/username/repo.git'
            }
        }

        stage('Run Unit Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        dir('backend') {
                            sh 'npm install'
                            sh 'npm test'
                        }
                    }
                }
                stage('Frontend Tests') {
                    steps {
                        dir('frontend') {
                            sh 'npm install'
                            sh 'npm test'
                        }
                    }
                }
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build Backend') {
                    steps {
                        script {
                            env.BACKEND_IMAGE = "yahyaguizani/minikubeimages:heart-backend-${params.BRANCH}-${env.BUILD_NUMBER}"
                            sh "docker build --cache-from $BACKEND_IMAGE -t $BACKEND_IMAGE ./backend"
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        script {
                            env.FRONTEND_IMAGE = "yahyaguizani/minikubeimages:heart-frontend-${params.BRANCH}-${env.BUILD_NUMBER}"
                            sh "docker build --cache-from $FRONTEND_IMAGE -t $FRONTEND_IMAGE ./frontend"
                        }
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                sh "docker push $BACKEND_IMAGE"
                sh "docker push $FRONTEND_IMAGE"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def KUBE_CTX = params.ENV == 'dev' ? env.KUBE_CONTEXT_DEV :
                                   params.ENV == 'staging' ? env.KUBE_CONTEXT_STAGING : env.KUBE_CONTEXT_PROD
                    sh "kubectl config use-context $KUBE_CTX"

                    sh "sed -i 's|yahyaguizani/minikubeimages:heart-backend|$BACKEND_IMAGE|' heart-backend.yaml"
                    sh "sed -i 's|yahyaguizani/minikubeimages:heart-frontend|$FRONTEND_IMAGE|' heart-frontend.yaml"

                    sh "kubectl apply -f heart-backend.yaml"
                    sh "kubectl apply -f heart-frontend.yaml"

                    try {
                        sh "kubectl rollout status deployment/heart-backend --timeout=120s"
                        sh "kubectl rollout status deployment/heart-frontend --timeout=120s"
                    } catch (err) {
                        echo "❌ Deployment failed, rolling back..."
                        sh "kubectl rollout undo deployment/heart-backend"
                        sh "kubectl rollout undo deployment/heart-frontend"
                        error "Deployment failed and rollback executed."
                    }
                }
            }
        }

        stage('Post Deployment Checks') {
            steps {
                sh 'kubectl get pods -o wide'
                sh 'kubectl get svc -o wide'
            }
        }
    }

    post {
        success {
            echo '✅ Deployment succeeded!'
            slackSend(channel: "$SLACK_CHANNEL", color: 'good', message: "✅ Deployment succeeded for build #${env.BUILD_NUMBER} on ${params.ENV}")
            mail to: 'team@example.com',
                 subject: "Deployment Success: Build #${env.BUILD_NUMBER} (${params.ENV})",
                 body: "Deployment was successful!\nImages: $BACKEND_IMAGE, $FRONTEND_IMAGE\nCheck Kubernetes services and pods."
        }
        failure {
            echo '❌ Deployment failed!'
            slackSend(channel: "$SLACK_CHANNEL", color: 'danger', message: "❌ Deployment failed for build #${env.BUILD_NUMBER} on ${params.ENV}")
            mail to: 'team@example.com',
                 subject: "Deployment Failed: Build #${env.BUILD_NUMBER} (${params.ENV})",
                 body: "Deployment failed!\nCheck Jenkins logs for details."
        }
    }
}
