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
        def buildLabel = ENV + "-" + currentBuild.number
        def jiraList = getJiraIssuesInCurrentBuild()
        addCommentsToJiraIssues(jiraList)
        addLabelsToJiraIssues(jiraList)
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

    def issue_pattern = "[Ss][Aa][Aa][Ss]-\\d+"
    def jiraList = []

    currentBuild.changeSets.each { changeSet ->
        changeSet.each { commit ->
            msg.findAll(issue_pattern).each { issue ->
                jiraList << [
                    issue: issue,
                    commit: commit
                ]
            }
        }
    }

    jiraList
}

def addCommentsToJiraIssues(jiraList) {
    jiraList.each { jira ->
        def ts = new Date(jira.commit.getTimestamp())
        String msg = jira.commit.getMsg()
        String comment = "[JENKINS-PIPELINE]\n" +
                         "Build: [" + stack + currentBuild.number + "|" + currentBuild.absoluteUrl + "]\n" +
                         "Result: " + currentBuild.result + "\n" +
                         "Commit Message: " + msg + "\n" +
                         "Author: " + jira.commit.getAuthor().getFullName() + " \n" +
                         "Timestamp: " + ts.format("MM-dd-yyyy HH:mm:ss", TimeZone.getTimeZone('UTC'))+" UTC\n"

        jiraAddComment(
            idOrKey: jira.issue,
            input: [ body: comment ],
            failOnError: false,
            auditLog: false
        )
    }
}

def addLabelsToJiraIssues(jiraList) {
    String jiraIssuesKeyList = ''
    jiraList.each { jira ->
        jiraIssueKeyList += ',' + jira.issue.key
    }

    def searchReults = jiraJqlSearch jql: "project = SAAS and issuekey in (" + jiraIssuesKeyList + ")"
    def issues = searchReults.data.issues
    def labels = []

    jiraList.each { jira ->
        issues.each { issue ->
            if (jira.issue == issue.key) {
                labels = issue.fields.labels
            }
        }

        labels << buildLabel

        jiraEditIssue(
            idOrKey: jira.issue,
            issue: [
                fields: [
                    project: [ key: 'SAAS' ],
                    labels: [ labels ]
                ]
            ]
        )
    }
}
