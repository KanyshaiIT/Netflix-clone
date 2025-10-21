pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'emilovaa'
        IMAGE_NAME = 'netflix-clone'
        VERSION = "v1"
        TMDB_V3_API_KEY = credentials('tmdb-api-key')  // Optional: store API key in Jenkins credentials
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📦 Cloning project..."
                checkout scm
                sh 'ls -l'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo "📥 Installing dependencies..."
                sh '''
                    if [ -f yarn.lock ]; then
                        yarn install
                    else
                        npm install
                    fi
                '''
            }
        }

        stage('Build React App') {
            steps {
                echo "🏗️ Building Netflix Clone app..."
                sh '''
                    if [ -f yarn.lock ]; then
                        yarn build
                    else
                        npm run build
                    fi
                '''
                sh 'ls -l dist || ls -l build'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh 'docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION} --build-arg TMDB_V3_API_KEY=${TMDB_V3_API_KEY} .'
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                echo "📤 Pushing image to Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        echo "$PASSWORD" | docker login -u "$USERNAME" --password-stdin
                        docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${VERSION}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "🚀 Deploying Netflix Clone to Kubernetes..."
                sh '''
                    kubectl apply -f Kubernetes/deployment.yml
                    kubectl apply -f Kubernetes/service.yml
                    sleep 10
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Netflix Clone deployed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check Jenkins logs for details."
        }
    }
}
