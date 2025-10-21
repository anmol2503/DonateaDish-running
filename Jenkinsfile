pipeline {
    agent any

    environment {
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/opt/homebrew/bin:${env.PATH}"
        DOCKER_IMAGE = "donateadish_app:latest"
        DOCKER_REGISTRY = "docker.io/anmol2503"
        ENV_FILE = ".env"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/anmol2503/DonateaDish-running.git'
            }
        }

        stage('Install & Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    # Optional: run tests
                    # pytest tests/
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            when {
                expression {
                    return env.SKIP_DOCKER_PUSH != 'true'
                }
            }
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                          usernameVariable: 'DOCKER_USER',
                                                          passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker tag $DOCKER_IMAGE $DOCKER_REGISTRY/$DOCKER_IMAGE
                                docker push $DOCKER_REGISTRY/$DOCKER_IMAGE
                                docker logout
                            '''
                        }
                    } catch (Exception e) {
                        echo "Failed to push Docker image: ${e.getMessage()}"
                        echo "Please ensure Docker Hub credentials are configured in Jenkins"
                        error("Docker push failed")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                        kubectl apply -f k8s/secrets.yaml
                        kubectl apply -f k8s/deployment.yaml
                        kubectl rollout status deployment/donateadish-app
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Please check the logs."
        }
    }
}
