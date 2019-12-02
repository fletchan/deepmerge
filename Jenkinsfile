pipeline {
  agent any
  tools { nodejs "node-12" }

  stages {
    stage('build') {
      steps {
        echo "Hello world"
        sh 'ls'
        echo "file path"
        echo "$BRANCH_NAME"
        sh 'pwd'
        sh 'which npm'
        sh 'which node'
        sh 'npm run build'
      }
    }

  }
}
