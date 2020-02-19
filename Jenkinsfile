pipeline {
  agent any
  stages {
    stage('build') {
      when {
        anyOf {
          branch master
          branch prod
        }
        beforeInput: true
      }
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
  tools {
    nodejs 'node-12'
  }
}

@NonCPS
def commentJiraIssues() {
    echo "commentJiraIssues"

    def issue_pattern = "[Ee][Pp][Oo]-\\d+"
    def jiraList = []

    currentBuild.changeSets.each { changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            msg.findAll(issue_pattern).each { id ->
                def ts = new Date(commit.getTimestamp())

                jiraList << [ id: id, comment: "[JENKINS-PIPELINE]\n" +
                                "Build: [Stack " + currentBuild.number + "|" + currentBuild.absoluteUrl + "] " + currentBuild.number + "\n" +
                                "Result: " + currentBuild.result + "\n" +
                                "Commit Message: " + msg + "\n" +
                                "Author: " + commit.getAuthor().getFullName() + " \n" +
                                "Timestamp: " + ts.format("MM-dd-yyyy HH:mm:ss", TimeZone.getTimeZone('UTC'))+" UTC\n"
                ]
            }
        }
    }

    echo "jiras: " + jiraList
    jiraList
}
