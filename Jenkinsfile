pipeline {
  agent any
  stages {
    stage('checkout') {
      steps('checkout') {
        sh 'pwd; ls;'
        checkout scm
        sh 'pwd; ls;'
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
      }
    }

  }
  tools {
    nodejs 'node-12'
  }
}
