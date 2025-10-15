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
                // Use GitHub credentials stored in Jenkins
                git branch: 'main',
                    url: 'https://github.com/anmol2503/DonateaDish-running.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                python3 -m venv venv
                source venv/bin/activate
                pip install --upgrade pip
                pip install -r requirements.txt
                # Optional: run tests if you have any
                # pytest tests/
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker --version
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
