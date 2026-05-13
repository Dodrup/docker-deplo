pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dodrup23/demo-jenkins-app"
    }

    stages {
        
        stage('Clone Repo') {
            steps {
                sh 'git clone https://github.com/dodrup23/docker-deplo.git'
            }
        }

        stage('Install Dependencies & Test') {
            steps {
                sh 'pip3 install flask pytest'
                
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to Docker') {
            steps {
                sh "docker rm -f demo-app || true"
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
    }
}
