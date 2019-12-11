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
        sh 'mkdir dbfx'
        sh 'cd dbfx'
        sh 'pwd'
        dir('dbfx') {
            git(url: 'ssh://buildsys@nj.dbnetworks.com:29418/dbfx', branch: 'master', credentialsId: 'buildsys-nj')
        }
      }
    }

  }
  tools {
    nodejs 'node-12'
  }
}
