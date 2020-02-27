def ENV;

pipeline {
  environment {
    JIRA_SITE = "jira"
    STACK = 'dev'
  }
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
        def buildLabel = env.STACK + "-" + currentBuild.number
        def jiraList = getJiraIssuesInCurrentBuild()
        echo "List " + jiraList
        echo "Stack " + env.STACK
        if (!jiraList.isEmpty()) {
          //addCommentsToJiraIssues(jiraList)
          echo "Calling into add labales"
          addLabelsToJiraIssues(jiraList)
        }
      }
    }
  }
  tools {
    nodejs 'node-12'
  }
}

@NonCPS
def getJiraIssuesInCurrentBuild() {
    echo "commentJiraIssues"

    def issue_pattern = "[Ee][Pp][Oo]-\\d+"
    def jiraList = []

    currentBuild.changeSets.each { changeSet ->
        changeSet.each { commit ->
            String msg = commit.getMsg()
            String author = commit.getAuthor().getFullName()
            def timestamp = new Date(commit.getTimestamp())
            msg.findAll(issue_pattern).each { issue ->
                jiraList << [
                    issue: issue,
                    commitMsg: msg,
                    commitAuthor: author,
                    commitTimestamp: timestamp
                ]
            }
        }
    }

    jiraList
}

def addCommentsToJiraIssues(jiraList) {
  echo "Stack inside addComments " + env.STACK
    jiraList.each { jira ->
        echo "What do we get for jira " + jira
        String comment = "[JENKINS-PIPELINE]\n" +
                         "Build: [" + env.STACK + currentBuild.number + "|" + currentBuild.absoluteUrl + "]\n" +
                         "Result: " + currentBuild.result + "\n" +
                         "Commit Message: " + jira.commitMsg + "\n" +
                         "Author: " + jira.commitAuthor + " \n" +
                         "Timestamp: " + jira.commitTimestamp.format("MM-dd-yyyy HH:mm:ss", TimeZone.getTimeZone('UTC'))+" UTC\n"

        jiraAddComment(
            idOrKey: jira.issue,
            input: [ body: comment ],
            failOnError: false,
            auditLog: false
        )

        echo "Done with one issue"
    }

    echo "Done iterating through adding comments"
}

def addLabelsToJiraIssues(jiraList) {
  def buildLabel = env.STACK + "-" + currentBuild.number

  echo "Adding labels function"
    def jiraIssuesKeyList = []
    jiraList.each { jira ->
        echo "Itererating through jira list" + jira
        jiraIssuesKeyList << "\"" + jira.issue + "\""
    }
  echo "jiraIssuesKeyList " + jiraIssuesKeyList
    String jiraIssuesSearchStr = jiraIssuesKeyList.join(",")
    echo "query string " + jiraIssuesSearchStr
    def searchResults = jiraJqlSearch jql: "project = EPO and issuekey in (" + jiraIssuesSearchStr + ")"
    echo "Search results " + searchResults
    def issues = searchResults.data.issues
    def labels = []

    jiraList.each { jira ->
        echo "\n jira: " + jira + "\n"
        issues.each { issue ->
            echo "\n issue: " + issue
            if (jira.issue == issue.key) {
                labels = issue.fields.labels
            }
        }
echo "Labels to add too " + labels
        labels << buildLabel
echo "Final labels " + labels
        jiraEditIssue(
            idOrKey: jira.issue,
            issue: [
                fields: [
                    project: [ key: 'SAAS' ],
                    labels: labels
                ]
            ]
        )
    }
}
