pipeline {
    agent any

    environment {
        // Updated with your repository name; change if needed
        DOCKER_IMAGE = "nikhil031020/devops-k8s-app"
        DOCKER_TAG   = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                echo 'Running application tests...'
                // Uses Windows 'if exist' instead of bash 'test -f'
                bat '''
                    if exist app\\index.html (
                        echo "Verification passed: app\\index.html exists."
                    ) else (
                        echo "Verification failed: app\\index.html not found."
                        exit /b 1
                    )
                '''
            }
        }

        stage('Docker Build') {
            steps {
                bat """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} -t ${DOCKER_IMAGE}:latest .
                """
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    // Windows echo piped into docker login
                    bat '''
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKER_USERNAME% --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                bat """
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                    kubectl --kubeconfig="C:\\Users\\Info\\.kube\\config" set image deployment/devops-k8s-app app=${DOCKER_IMAGE}:${DOCKER_TAG} -n development
                """
            }
        }

        stage('Verify Deployment') {
            steps {
                bat """
                    kubectl --kubeconfig="C:\\Users\\Info\\.kube\\config" rollout status deployment/devops-k8s-app -n development
                """
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully!'
        }
        failure {
            echo 'CI/CD pipeline failed!'
        }
    }
}
