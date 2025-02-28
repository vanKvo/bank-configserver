pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "dolphin2002/configserver:main"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')
    }

    stages {
        stage('Build & Test') {
            when {
                branch 'main'
            }
            steps {
                // Compile, test, and package the code.
                sh 'mvn clean package'
            }
        }

        stage('Build Docker Image') {
            steps {

                    //sh "docker build -t ${DOCKER_IMAGE}:latest ."
                    sh 'mvn compile jib:dockerBuild'
            }
        }

        stage('Login into Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-token', passwordVariable: 'DOCKERHUB_PSW', usernameVariable: 'DOCKERHUB_USR')]) {
                    sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}"
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker stop configserver-container || true
                docker rm configserver-container || true
                docker run -d --name configserver-container -p 8071:8071 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}
