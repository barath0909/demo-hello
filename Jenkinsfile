pipeline {
    agent any

    environment {
        IMAGE_NAME = "barathj09/demohello"
        DEPLOYMENT_FILE = "deployment.yaml"
    }

    stages {

        stage('Git Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Generate Image Tag') {
            steps {
                script {
                    env.IMAGE_TAG = new Date().format("yyyy-MM-dd-HHmmss")
                    env.FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"

                    echo "Image Name: ${FULL_IMAGE}"
                }
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker_cred',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${FULL_IMAGE} .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                sh '''
                docker push ${FULL_IMAGE}
                '''
            }
        }

        stage('Update Kubernetes Deployment File') {
            steps {
                sh """
                sed -i 's|image: .*|image: ${FULL_IMAGE}|' ${DEPLOYMENT_FILE}

                git config user.name "Jenkins"
                git config user.email "jenkins@example.com"

                git add ${DEPLOYMENT_FILE}
                git commit -m "Updated image to ${FULL_IMAGE}" || echo "No changes to commit"

                git push
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f deployment.yaml
                '''
            }
        }

    }

    post {

        success {
            echo "Pipeline executed successfully"
        }

        failure {
            echo "Pipeline failed"
        }

    }
}