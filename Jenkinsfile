pipeline {
  agent any
  stages {
    stage('build') {
      steps {
        echo 'Hello world'
        sh 'ls'
        echo 'file path'
        echo "$BRANCH_NAME"
        sh 'pwd'
        sh 'which npm'
        sh 'which node'
        git(url: 'ssh://buildsys@nj.dbnetworks.com:29418/dbfx', branch: 'master')
      }
    }

  }
  tools {
    nodejs 'node-12'
  }
}