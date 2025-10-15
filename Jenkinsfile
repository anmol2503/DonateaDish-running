pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "donateadish_app:latest"
        DOCKER_REGISTRY = "docker.io/anmol2503"
        ENV_FILE = ".env"
    }

    stages {

        stage('Checkout') {
            steps {
                // Use your GitHub repo URL
                git branch: 'main', url: 'https://github.com/anmol2503/DonateaDish-running.git'
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                # Use Docker Python image to ensure correct Python version
                docker run --rm -v $PWD:/app -w /app python:3.12-slim bash -c "
                    pip install --upgrade pip &&
                    pip install -r requirements.txt
                    # Optional: run tests if you have any
                    # pytest tests/
                "
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker tag $DOCKER_IMAGE $DOCKER_REGISTRY/$DOCKER_IMAGE
                    docker push $DOCKER_REGISTRY/$DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                # Apply your Kubernetes manifests
                kubectl apply -f k8s/
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}
