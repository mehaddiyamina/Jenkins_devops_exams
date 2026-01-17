pipeline {
  agent any

  environment {
    // identifiant Jenkins Credentials (Username + Password/PAT)
    DOCKERHUB_CREDS = credentials('dockerhub-creds')

    // ton dockerhub username
    DOCKERHUB_USER = "yamina99"

    // helm chart
    CHART_PATH   = "./charts"
    RELEASE_NAME = "jenkins-exam"
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'master',
          url: 'https://github.com/mehaddiyamina/Jenkins_devops_exams.git'
      }
    }

    // ✅ IMPORTANT: login AVANT build, sinon docker ne peut pas pull python:3.8-slim
    stage('DockerHub login (before build)') {
      steps {
        sh '''
          set -e
          echo "$DOCKERHUB_CREDS_PSW" | docker login -u "$DOCKERHUB_USER" --password-stdin
          docker info >/dev/null
        '''
      }
    }

    stage('Build images') {
      steps {
        sh '''
          set -e
          docker-compose build
        '''
      }
    }

    stage('Push images') {
      steps {
        sh '''
          set -e
          docker-compose push
        '''
      }
    }

    stage('Deploy DEV') {
      steps {
        sh '''
          set -e
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          helm upgrade --install ${RELEASE_NAME}-dev ${CHART_PATH} --namespace dev --create-namespace
        '''
      }
    }

    stage('Deploy QA') {
      steps {
        sh '''
          set -e
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          helm upgrade --install ${RELEASE_NAME}-qa ${CHART_PATH} --namespace qa --create-namespace
        '''
      }
    }

    stage('Deploy STAGING') {
      steps {
        sh '''
          set -e
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          helm upgrade --install ${RELEASE_NAME}-staging ${CHART_PATH} --namespace staging --create-namespace
        '''
      }
    }

    stage('Deploy PROD (manual)') {
      when { expression { return false } } // tu l'actives manuellement si l'exam le demande
      steps {
        sh '''
          set -e
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          helm upgrade --install ${RELEASE_NAME}-prod ${CHART_PATH} --namespace prod --create-namespace
        '''
      }
    }
  }

  post {
    always {
      sh 'docker logout || true'
    }
  }
}

