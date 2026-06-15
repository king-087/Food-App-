pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "karthi087/food-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Test Docker Image') {
           steps {
               sh '''
                   docker run -d --name food-test -p 8085:80 ${DOCKER_IMAGE}:${DOCKER_TAG}

                   sleep 20

                   curl -f http://localhost:8085

                   docker rm -f food-test
               '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}

                        docker push ${DOCKER_IMAGE}:latest
                    '''
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                sh '''
                    kubectl apply -f k8s/

                    kubectl rollout status deployment/food-app --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get deployments
                    kubectl get pods
                    kubectl get svc
                '''
            }
        }
    }

    post {
        success {
            echo "SUCCESS: Food-App deployed successfully to AKS"
        }

        failure {
            echo "FAILED: Check Jenkins Console Output"
        }

        always {
            sh 'docker rm -f food-test || true'
        }
    }
}
