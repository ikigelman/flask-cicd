
pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'iritkigelman/flask-cicd'
        DOCKER_TAG = '1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-flask-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(
                    credentialsId: 'k3s-kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        echo "Deploying Flask to Kubernetes"

                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml

                        kubectl rollout status deployment/flask-deployment --timeout=180s

                        kubectl get pods
                        kubectl get services
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                withCredentials([file(
                    credentialsId: 'k3s-kubeconfig',
                    variable: 'KUBECONFIG'
                )]) {
                    sh '''
                        echo "Test 1: Testing Flask through Kubernetes Service"

                        RESPONSE=$(kubectl exec deployment/flask-deployment -- python -c "import urllib.request; print(urllib.request.urlopen('http://flask-service:5000').read().decode())")

                        echo "$RESPONSE"

                        test "$RESPONSE" = "Hello World from Flask!"

                        echo "Test 1 PASSED"

                        echo "Test 2: Testing Flask through Traefik LoadBalancer"

                        RESPONSE=$(python3 -c "import urllib.request; print(urllib.request.urlopen('http://192.168.1.250/flask', timeout=10).read().decode())")

                        echo "$RESPONSE"

                        test "$RESPONSE" = "Hello World from Flask!"

                        echo "Test 2 PASSED"
                    '''
                }
            }
        }
    }
}
