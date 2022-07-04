---
title: Parsing data between Jenkins Jobs
description: >-
  Jenkins is the most widely used CICD/Build tool that’s being used for many
  software projects. The stability and the extensibility of…
date: '2019-05-20T13:43:28.538Z'
categories: [Devops, CICD, Jenkins]
tags: [jenkins, cicd, devops]
keywords: []
image:
  path: /assets/img/medium/0__Mgvcsevtq9ZbvROi.jpg
  width: 800
  height: 500
  alt: Amazon AWS
---

Jenkins is the most widely used CICD/Build tool that’s being used for many software projects. The stability and the extensibility of Jenkins has made it the number one go to tool when doing CICD work.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__PFUD__bMh__K6EzCvCXFY__vQ.png)

In this post I will explain how you can parse some information from one build to the next build. My use case was, I was creating docker images with a build pipeline to update an environment and after updating a environment if something goes wrong in the environment the user has the ability to revert the environment. So inorder to rever the environment the build pipeline had to know the currently deployed versions.

This could be simply done with some groovy code. In this case I was using Env Inject plugin and I updated the Environment variable with the information I needed for the next job and this env variable was read by the next Job. Following is the simple groovy code you can use for this.

```groovy
import org.jenkinsci.plugins.envinject.EnvInjectJobProperty
import org.jenkinsci.plugins.envinject.EnvInjectJobPropertyInfo

node('master') {
    script {
        echo 'Hello World'
        echo "$JOB_NAME"
        echo "$perform"
        persistInfo();
    }
}

def persistInfo(){

    def jenkins = Jenkins.instance
    def jobA = jenkins.getItemByFullName("$JOB_NAME")
    def prop2 = jobA.getProperty(org.jenkinsci.plugins.envinject.EnvInjectJobProperty);
    def con = prop2.getInfo().getPropertiesContent();

    def orig = []
    def str = ""
    for(e in con.split(System.getProperty("line.separator"))) {
        if (!e.contains("UPDATED_DOCKER_IMAGES")) {
            str = str + e + System.getProperty("line.separator")
        }
    }
    str = str + "UPDATED_DOCKER_IMAGES=wso2apim-2.6.0"
    
    def prop = new EnvInjectJobPropertyInfo("", str , "", "", "", false)

    def propNew = new org.jenkinsci.plugins.envinject.EnvInjectJobProperty(prop)
    propNew.setOn(true)
    propNew.setKeepBuildVariables(true)
    propNew.setKeepJenkinsSystemVariables(true)
    jobA.addProperty(propNew)

    jobA.save();
}
```

What the above will do is read the existing environment variables and add a new environment variable called UPDATED\_DOCKER\_IMAGES. This property can be read from the next build.

Hope this helps someone in need. Please drop a comment if you have any queries.