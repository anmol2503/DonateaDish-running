pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "donateadish_app:latest"
        DOCKER_REGISTRY = "docker.io/anmol2503"
        ENV_FILE = ".env"
        PATH = "/opt/homebrew/bin:${env.PATH}"  // ensure Jenkins sees docker
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/anmol2503/DonateaDish-running.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                                                  passwordVariable: 'DOCKER_PASS', 
                                                  usernameVariable: 'DOCKER_USER')]) {
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
                sh 'kubectl apply -f k8s/'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished"
        }
    }
}
