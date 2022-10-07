---
title: Stop A Recurring Jenkins Job After a Fixed number of Runs
description: Stop A Recurring Jenkins Job After a Fixed number of Runs
date: '2022-10-06'
categories: [Automation, CICD]
keywords: [jenkins, cicd, devops, groovy, automation]
tags: [jenkins, cicd, devops, groovy, automation]
image:
  path: /assets/img/posts/jenkins.jpg
  width: 800
  height: 500
  alt:
---

This solution was inspired by a Stackoverflow question I happen to answer. The requirement is not very common but thought of sharing it if anyone ever needs it. The requirement is something like this, once a Jenkins Job is triggered manually, it should run X number of builds, running one build every day and then stop after X number of runs. There is no such functionality in Jenkins to support this kind of behavior OOB, so how can we achieve this? On top of my head, I thought maybe we can play with the Cron expression configured in a Job dynamically to run and then stop. So what I ended up doing is adding a Cron expression to run every day if manually triggered and then removing the corn expression after a predefined number of runs elapse. The next problem is how would we determine how many runs it ran. Jenkins doesn't have an OOB way to persist and pass Build data between different builds, so the easiest way to get around this is to check each build before the current build and determine whether it has executed X number of Build by a Timer Trigger. When a build is triggered in Jenkins, it's associated with a cause. In other words, how the Build was triggered. For example, you may have, "User Triggers", "Timer Triggers", "Remote Triggers" etc. so you should be able to retrieve this information and use them in your flow.

Let's come up with a Groovy function we can use within the Pipeline to return the correct Cron expression based on the number of runs and the type of trigger.

```groovy
def getCron() {
    
    def runEveryDayCron = "0 9 * * *" //Runs everyday at 9
    def numberOfRunsToCheck = 7 // Will run 7 times
    
    def currentBuildNumber = currentBuild.getNumber()
    def job = Jenkins.getInstance().getItemByFullName(env.JOB_NAME)
   
    for(int i=currentBuildNumber; i > currentBuildNumber - numberOfRunsToCheck; i--) {
        def build = job.getBuildByNumber(i)
        if(build.getCause(hudson.model.Cause$UserIdCause) != null) { //This is a manually triggered Build
            return runEveryDayCron
        }
    }
    return ""
}
```
The above function will traverse through the past builds and determine whether it has elapsed the expected number of Builds, if so the Cron expression will be set to blank preventing it from running again. Once you execute the Build again(Manually) the Cron expression will be set to run every day. The logic is pretty simple.

Next, let's set the Con expression to the Pipeline. Check the following complete Pipeline with the Groovy function embedded. Once executed the correct Cron will be retrieved and set to the Pipeline.

```groovy
def expression = getCron()

pipeline {
    agent any
    triggers{ cron(expression) }
    stages {
        stage('Example') {
            steps {
                script {
                    echo "Build"
                }
           }
        }
    }
}

def getCron() {
    
    def runEveryDayCron = "0 9 * * *" //Runs everyday at 9
    def numberOfRunsToCheck = 7 // Will run 7 times
    
    def currentBuildNumber = currentBuild.getNumber()
    def job = Jenkins.getInstance().getItemByFullName(env.JOB_NAME)
   
    for(int i=currentBuildNumber; i > currentBuildNumber - numberOfRunsToCheck; i--) {
        def build = job.getBuildByNumber(i)
        if(build.getCause(hudson.model.Cause$UserIdCause) != null) { //This is a manually triggered Build
            return runEveryDayCron
        }
    }
    return ""
}
```

Hope the above helps. Drop a comment if you have any questions. 