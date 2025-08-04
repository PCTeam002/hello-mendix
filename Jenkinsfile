pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'pyndhotaraya/hello-mendix:latest'
    }
    stages {
        stage('Checkout Code') {
            steps {
                echo "📥 Checking out source code from GitHub..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Building Docker image: ${DOCKER_IMAGE}"
                    // Build dari root repo agar folder src/ terbaca
                    sh 'docker build -t $DOCKER_IMAGE -f deployment/Dockerfile .'
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "🔑 Logging in to DockerHub and pushing image..."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login',
                        usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push $DOCKER_IMAGE'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-cred', variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f deployment/kubernetes/deployment.yaml'
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build & deployment successful!"
        }
        failure {
            echo "❌ Build or deployment failed! Check logs above for details."
        }
    }
}
