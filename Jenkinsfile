stack = ""

pipeline {
  environment {
    JIRA_SITE = "jira"
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
          if (env.GIT_BRANCH == 'master') {
            echo "Inside of if master"
            stack = 'dev'
            echo "Right after set: ${stack}"
          }

          echo "Build branch: " + env.GIT_BRANCH
          echo "Stack: " + stack
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
        testFunction()
        if (!jiraList.isEmpty()) {
          //addCommentsToJiraIssues(jiraList)
          //addLabelsToJiraIssues(jiraList)
        }

        sh("git config user.name 'buildsys'")
        sh("git config user.email 'buildsys@dbnetworks.com'")
        sh("git tag " + buildLabel)
        sh("git push --tags")

      }
    }
  }
  tools {
    nodejs 'node-12'
  }
}

def testFunction() {
  echo "Stack: " + stack
  return
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

        if (labels.isEmpty()) {
          echo "skipping loop"
          return;
        }

        echo "Do we get here after empty check"

        labels << buildLabel

        /*jiraEditIssue(
            idOrKey: jira.issue,
            issue: [
                fields: [
                    project: [ key: 'SAAS' ],
                    labels: labels
                ]
            ]
        )*/
    }
}
