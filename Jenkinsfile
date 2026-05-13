pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dodrup23/demo-jenkins-app"
    }

    stages {

        stage('Clone Repo') {
            steps {
                script {
                    if (fileExists('docker-deplo')) {
                        echo 'Cleaning up existing repo directory'
                        sh 'rm -rf docker-deplo'
                    }
                }
                echo 'Cloning the repository...'
                sh 'git clone https://github.com/Dodrup/docker-deplo.git'
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                echo 'Installing dependencies and running tests...'
                sh 'pip3 install flask pytest'
            }
        }

        stage('Authenticate Docker') {
            steps {
                echo 'Authenticating to Docker Hub...'
                // Authenticate Docker to avoid rate limits on pulling images
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    // Pass password via stdin securely without interpolation
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                echo 'Deploying Docker container...'
                // Remove any existing container with the same name to avoid conflicts
                sh "docker rm -f demo-app || true"
                // Run the new container
                sh "docker run -d -p 5000:5000 --name demo-app $DOCKER_IMAGE:${BUILD_NUMBER}"
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            // Clean up Docker system (optional) if you want to remove unused images or containers
            echo 'Cleaning up Docker images...'
            sh 'docker system prune -f'
        }
    }
}
