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
    stage('comment jira') {
        steps {
          script {
            commentJiraIssues().each { jiraChange ->
                echo "jiraChange:  " + jiraChange

                jiraAddComment(
                    idOrKey: jiraChange.id,
                    comment: jiraChange.comment,
                    failOnError: false
                )
            }
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

    def issue_pattern = "[Ss][Aa][Aa][Ss]-\\d+"
    def jiraList = []

    currentBuild.changeSets.each { changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            msg.findAll(issue_pattern).each { id ->
                def ts = new Date(commit.getTimestamp())

                jriaList << [ id: id, comment: "[JENKINS-PIPELINE]\n" +
                                "Current Build: " + currentBuild.absoluteUrl + "\n" +
                                "Commit Message: " + msg + "\n" +
                                "Author: " + commit.getAuthor().getFullName() + "\n" +
                                "Timestamp: " + ts.format("yyyyMMdd.HHmmss", TimeZone.getTimeZone('UTC'))+" UTC\n"
                ]
            }
        }
    }

    echo "jiras: " + jiraList
    jiraList
}
