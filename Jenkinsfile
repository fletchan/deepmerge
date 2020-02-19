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
      }
    }

    stage('test') {
      steps {
        waitUntil() {
          input 'wait'
        }

      }
    }

  }
  tools {
    nodejs 'node-12'
  }
  post {
    always {
      script {
        commentJiraIssues().each { jiraChange ->
        echo "jiraChange: " + jiraChange

        jiraAddComment(
          idOrKey: jiraChange.id,
          input: [ body: jiraChange.comment ],
          failOnError: false,
          site: "jira",
          auditLog: false
        )
      }
    }

  }

}
}