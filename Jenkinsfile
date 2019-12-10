pipeline {
  agent any
  stages {
    stage('checkout') {
      steps('checkout') {
        checkout scm
      }
    }
    stage('build') {
      steps {
        echo 'Hello world'
        sh 'ls'
        echo 'file path'
        echo "$BRANCH_NAME"
        sh 'pwd'
        sh 'which npm'
        sh 'which node'
        git(url: 'ssh://buildsys@nj.dbnetworks.com:29418/dbfx', branch: 'master', credentialsId: 'buildsys-nj')
      }
    }

  }
  tools {
    nodejs 'node-12'
  }
}