pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "koushikbijili/cicd-node-app"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}")
                    docker.build("${DOCKERHUB_REPO}:latest")
                }
            }
        }

        stage('Login to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh "docker push ${DOCKERHUB_REPO}:${env.BUILD_NUMBER}"
                sh "docker push ${DOCKERHUB_REPO}:latest"
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                docker stop cicd-app || true
                docker rm cicd-app || true
                docker run -d -p 80:80 --name cicd-app ${DOCKERHUB_REPO}:latest
                '''
            }
        }
    }
}
