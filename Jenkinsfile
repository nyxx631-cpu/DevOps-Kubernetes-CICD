pipeline {

    agent any

    environment {

        DOCKER_IMAGE = "nikhil031020/devops-k8s-app"
        DOCKER_TAG = "${BUILD_NUMBER}"

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

                sh '''
                    test -f app/index.html
                '''
            }
        }

        stage('Docker Build') {

            steps {

                sh """
                    docker build \
                    -t ${DOCKER_IMAGE}:${DOCKER_TAG} \
                    -t ${DOCKER_IMAGE}:latest \
                    .
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

                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                        -u "$DOCKER_USERNAME" \
                        --password-stdin
                    '''
                }
            }
        }

        stage('Docker Push') {

            steps {

                sh """
                    docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Deploy to Kubernetes') {

            steps {

                sh """
                    kubectl set image deployment/devops-k8s-app \
                    app=${DOCKER_IMAGE}:${DOCKER_TAG} \
                    -n development
                """
            }
        }

        stage('Verify Deployment') {

            steps {

                sh """
                    kubectl rollout status \
                    deployment/devops-k8s-app \
                    -n development
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
