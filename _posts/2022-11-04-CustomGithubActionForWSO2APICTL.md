---
title: Custom Github Action for WSO2 APICTL
description: Custom Github Action for WSO2 APICTL
date: '2022-11-04'
categories: [Automation]
keywords: []
tags: [Github, cicd, wso2, automation]
image:
  path: /assets/img/posts/GHAction.png
  width: 800
  height: 500
  alt:
---

I have worked with Github Actions in some projects and used quite a few Actions written by others. So I wanted to write my own custom action to experience the process. So here I'm with this post after building a custom GitHUb Action to setup WSO2 APICTL. This post will not explain how to build a custom action but rather how to use the custom action that was built. The custom Action is located in the Guthub Market place at https://github.com/marketplace/actions/setup-wso2-apictl

### What is WSO2 API CTL

WSO2 API Controller (apictl) is a command-line tool providing the capability to move APIs, API Products, and Applications across environments and to perform CI/CD operations. Furthermore, it can perform WSO2 Micro Integrator (WSO2 MI) server specific operations such as monitoring Synapse artifacts and performing MI management/administrative tasks from the command line.

### Why the Custom Action

If you are working on Github and using Github workflows to manage your deployments and to develop CICD pipelines, for any CICD pipeline related WSO2 APIM or MI this action can be used. This custom GH action will basically setup WSO2 APICTL in a Linux environment which can be used to perform different operations like, moving APIs between environments etc. Since there are no usecases for setting up on Windows environments, currently only Linux environments are supported.

### How to use the Action

Basically in your workflow you can use the action by adding the following step. (Make sure you get the latest version from Github)

```yaml
uses: yasassri/setup-wso2-apictl@v1.2
with:
  version: '4.1.0'
```

You are setup to any version of the APICTL by passing the version parameter. Or If you wish to install APICTL by providing a URL to a tarball location, that will also work. Following are the parameters you can pass as inputs. 

**`version`**

[Optional] The version of the APICTL to setup. The default vale is `"4.1.0"`

The version will be picked from the Github releases at https://github.com/wso2/product-apim-tooling/releases. 
Ex: 4.1.0, v4.1.0, 3.2.5 etc.

**`tarball_location`**

[Optional] A location to an APICTL Tarball if it needs to be downloaded from a custom location. No Default

### Examples of different inputs


```yaml
uses: yasassri/setup-wso2-apictl@v1.2
with:
  version: '4.1.0'
```

```yaml
uses: yasassri/setup-wso2-apictl@v1.2
with:
  tarball_location: 'https://github.com/wso2/product-apim-tooling/releases/download/v4.1.0/apictl-4.1.0-linux-x64.tar.gz'
```

### Full workflow

Following is a full workflow with the custom action.

```yaml
name: WSO2_APIM
on: [push]

jobs:
  wso2:
    runs-on: ubuntu-latest
    permissions:
      issues: write
      pull-requests: write
    steps:
    - uses: yasassri/setup-wso2-apictl@v1.2
      with:
        version: 'v3.2.5'
    - run: apictl version
```
