pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t flask-hello-world:1.0 .'
            }
        }

        stage('Push') {
            steps {
                echo 'Pushing Docker image'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying to Kubernetes'
            }
        }
    }
}
