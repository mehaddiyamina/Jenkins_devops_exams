pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKERHUB_USER = "${DOCKERHUB_CREDENTIALS_USR}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/mehaddiyamina/Jenkins_devops_exams.git'
            }
        }

        stage('Build images') {
            steps {
                sh 'docker-compose build'
            }
        }

        stage('Login DockerHub') {
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_USER --password-stdin
                '''
            }
        }

        stage('Push images') {
            steps {
                sh 'docker-compose push'
            }
        }
    }
}

