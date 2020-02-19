def ENV;

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
        script {
          ENV = 'test'
        }
      }
    }
  }
  post {
    always {
      script {
        def buildLabel = ENV + currentBuild.number

        commentJiraIssues().each { jiraChange ->
            echo "jiraChange: " + jiraChange

            jiraAddComment(
                idOrKey: jiraChange.id,
                input: [ body: jiraChange.comment ],
                failOnError: false,
                site: "jira",
                auditLog: false
            )

            def issueUpdate = [ fields: [
              project: [ key: 'EPO' ],
              labels: [ buildLabel ]
            ]]
            def queryParams = [notifyUsers: true]

            response = jiraEditIssue idOrKey: jiraChange.id, queryParams: queryParams, issue: issueUpdate, site: "jira"

            echo response.successful.toString()
            echo response.data.toString()
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
