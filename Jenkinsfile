pipeline {
  agent any

  environment {
    // Jenkins credentials (type: Username with password)
    DOCKERHUB_CREDS_ID = 'dockerhub-creds'

    // A REMPLACER par ton username DockerHub (souvent = ton username GitHub)
    DOCKERHUB_USER = 'mehaddiyamina'

    // A REMPLACER : nom de release Helm (peut être ce que tu veux)
    RELEASE_NAME = 'jenkins-exam'

    // Path chart (ici ton chart est dans ./charts)
    CHART_PATH = './charts'
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
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

    stage('DockerHub login') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDS_ID}", usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh '''
            set -e
            echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
          '''
        }
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
          helm upgrade --install ${RELEASE_NAME}-dev ${CHART_PATH} \
            --namespace dev --create-namespace
        '''
      }
    }

    stage('Deploy QA') {
      steps {
        sh '''
          set -e
          helm upgrade --install ${RELEASE_NAME}-qa ${CHART_PATH} \
            --namespace qa --create-namespace
        '''
      }
    }

    stage('Deploy STAGING') {
      steps {
        sh '''
          set -e
          helm upgrade --install ${RELEASE_NAME}-staging ${CHART_PATH} \
            --namespace staging --create-namespace
        '''
      }
    }

    stage('Deploy PROD (manual)') {
      when { branch 'master' }
      steps {
        input message: "Valider le déploiement PROD ?", ok: "DEPLOY"
        sh '''
          set -e
          helm upgrade --install ${RELEASE_NAME}-prod ${CHART_PATH} \
            --namespace prod --create-namespace
        '''
      }
    }
  }
}

