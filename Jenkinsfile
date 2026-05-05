pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'nadzalla'
        DOCKER_HUB_ID   = 'dockerhub-login'
        GIT_REPO_URL    = 'https://github.com/nadzallad/Games.git'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: "${GIT_REPO_URL}"
            }
        }

        stage('Build & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_HUB_ID}",
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {

                    sh 'echo $PASS | docker login -u $USER --password-stdin'

                    sh "docker build -t $DOCKER_HUB_USER/games-backend:latest ./backend"
                    sh "docker build -t $DOCKER_HUB_USER/games-frontend:latest ./frontend"

                    sh "docker push $DOCKER_HUB_USER/games-backend:latest"
                    sh "docker push $DOCKER_HUB_USER/games-frontend:latest"
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([file(credentialsId: 'aks-config', variable: 'KUBECONFIG_FILE')]) {

                    sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f games-k8s.yaml'
                    sh 'kubectl --kubeconfig=$KUBECONFIG_FILE apply -f games-ingress.yaml'
                }
            }
        }
    }
}