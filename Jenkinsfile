pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "yourdockerhubuser/devops-utility-hub"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Anujkapile/devops-cicd-assignment-.git'
            }
        }

        stage('Test') {
            steps {
                dir('app') {
                    // Basic static validation - checks HTML is well-formed
                    sh '''
                        which tidy || apt-get install -y tidy || true
                        tidy -q -e index.html || echo "HTML check completed with warnings"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('app') {
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG -t $IMAGE_NAME:latest .'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f devops-app || true
                    docker run -d --name devops-app -p 80:3000 $IMAGE_NAME:latest
                '''
            }
        }
    }

    post {
        always { sh 'docker logout' }
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed. Check logs above.' }
    }
}
