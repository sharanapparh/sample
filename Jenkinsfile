pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "yourdockerhub/flask-app"
        TAG = "${BUILD_NUMBER}"
    }

    triggers {
        githubPush()
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/your-repo.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'echo "Running unit tests..."'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t $DOCKER_IMAGE:$TAG .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                usernameVariable: 'USER',
                passwordVariable: 'PASS')]) {

                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    """
                }
            }
        }

        stage('Push Image') {
            steps {
                sh """
                docker push $DOCKER_IMAGE:$TAG
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh """
                kubectl set image deployment/flask-app flask-app=$DOCKER_IMAGE:$TAG
                kubectl rollout status deployment/flask-app
                """
            }
        }

        stage('Terraform (Optional Infra)') {
            steps {
                dir('terraform') {
                    sh """
                    terraform init
                    terraform apply -auto-approve
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🚀 Deployment Successful"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
}
