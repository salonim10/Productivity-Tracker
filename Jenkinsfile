
pipeline {
    agent any

    environment {
        IMAGE_REPO = 'salonimuley/productivity-tracker'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Python Validation') {
            steps {
                sh '''
                    python3 -m compileall -q .
                    if [ -d tests ]; then
                        python3 -m unittest discover -s tests -v
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_REPO}:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_PASS'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_PASS" | docker login \
                            -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${IMAGE_REPO}:${BUILD_NUMBER}
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kubectl apply -f k8s/
                    kubectl set image deployment/productivity-tracker \
                        productivity-tracker=${IMAGE_REPO}:${BUILD_NUMBER}
                    kubectl rollout status \
                        deployment/productivity-tracker --timeout=180s
                    kubectl get pods,services
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
