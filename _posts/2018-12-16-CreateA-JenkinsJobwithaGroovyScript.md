---
title: Create A Jenkins Job with a Groovy Script
description: >-
  In this post I will share some useful groovy code that can be used when
  creating a Jenkins job using a groovy script, so this groovy…
date: '2018-12-16T15:27:51.061Z'
categories: [jenkins, cicd, devops]
keywords: []
image:
  path: /assets/img/medium/0__vJucPzrIyJ0ojuzs.jpg
  width: 800
  height: 500
  alt: 
---

In this post I will share some useful groovy code that can be used when creating a Jenkins job using a groovy script, so this groovy script can be used within a declarative pipeline as well if you wish to create Jobs dynamically from within an another job. The simple code is as follows, I have added few inline comments so the code snippet will be easy to understand.

```groovy
/**
 * This method is responsible for creating the Jenkins job.
 * @param jobName jobName
 * @param timerConfig cron expression to schedule the job
 * @return
 */
def createJenkinsJob(def jobName, def timerConfig) {

    echo "Creating the job ${jobName}"
  // Here I'm using a shared library in the pipeline, so I have loaded my shared library here
  // You can simply have the entire pipeline syntax here.
    def jobDSL="@Library('yasassri@master') _\n" +
                "Pipeline()"
    def flowDefinition = new org.jenkinsci.plugins.workflow.cps.CpsFlowDefinition(jobDSL, true)
    def instance = Jenkins.instance
    def job = new org.jenkinsci.plugins.workflow.job.WorkflowJob(instance, jobName )
    job.definition = flowDefinition
    job.setConcurrentBuild(false)

  // Adding a cron configurations if cron configs are provided
    if (timerConfig != null && timerConfig != "") {
        hudson.triggers.TimerTrigger newCron = new hudson.triggers.TimerTrigger(timerConfig);
        newCron.start(job, true)
        job.addTrigger(newCron)
    }
  // Here I'm adding a Job property to the Job, i'm using environment inject plugin here
    def rawYamlLocation = "http://localhost:80/text.yaml"
    def prop = new EnvInjectJobPropertyInfo("", "KEY=${rawYamlLocation}", "",
            "", "", false)
    def prop2 = new org.jenkinsci.plugins.envinject.EnvInjectJobProperty(prop)
    prop2.setOn(true)
    prop2.setKeepBuildVariables(true)
    prop2.setKeepJenkinsSystemVariables(true)
    job.addProperty(prop2)
    job.save()
    Jenkins.instance.reload()
}
```

Hope the above was useful, please drop a comment if you have any concerns.