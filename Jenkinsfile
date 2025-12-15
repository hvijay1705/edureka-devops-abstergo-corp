pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = 'edureka-dockerhub-cred'
        DOCKERHUB_REPO = 'docker.io/hvijay1705/edureka-devops-abstergo-corp'
        K8S_NAMESPACE = 'abstergo'
        APP_NAME = 'edureka-devops-abstergo-corp'
    }

    triggers {
        // Poll GitHub every 2 minutes (simpler than webhooks for demo)
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker build -t ${DOCKERHUB_REPO}:latest .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}",
                                                usernameVariable: 'DOCKER_USER',
                                                passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push ${DOCKERHUB_REPO}:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    # Update image in deployment
                    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKERHUB_REPO}:latest -n ${K8S_NAMESPACE} || \
                    kubectl apply -f k8s/deployment.yaml

                    # Ensure service exists
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}