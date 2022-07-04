---
title: >-
  Building Your WSO2 Integrations with Unit Tests; A Guide for WSO2 Micro
  Integrator & Enterprise…
description: >-
  WSO2 Micro Integrator(MI) and Enterprise Integrator(EI) is a feature rich
  Opensource Integration platform which provides traditional…
date: '2021-03-31T05:02:13.195Z'
categories: []
keywords: []
tags: [java, aws]
image:
  path: /assets/img/medium/0__Mgvcsevtq9ZbvROi.jpg
  width: 800
  height: 500
  alt:
---

![](/assets/img/medium/0__UfYm4efE__QTONlm5.jpg)

WSO2 Micro Integrator(MI) and Enterprise Integrator(EI) is a feature rich Opensource Integration platform which provides traditional Enterprise Service Bus capabilities combined with modern cutting edge features. Developing and deploying integration use cases with WSO2 MI/EI is similar to developing any kind of application, which follows a similar lifecycle. When developing an integration use case, you have the source code, which consist of unit tests and when the source is compiled, the unit tests are executed and it produces a deployable artefact. In this post I will unveil few strategies you can adopt when running unit tests for your integrations.

As mentioned WSO2 Micro Integrator and Enterprise Integrator provides the capability to the users to write unit tests to make sure the integration are working as expected. In order to develop integrations the users can use Integration Studio which is an Eclipse based IDE capable of providing users with Drag and drop capabilities as well as source editing capabilities. Also it allows you to write Unit tests and to run these tests easily using the Integrations Studio. The major focus of this post is to discuss how can you execute these tests using maven when the artefacts are built in a CICD pipeline. I’m assuming you already know how to write unit tests in WSO2 Integrations Studio and know basic build commands with Maven.

Let’s get started :)

#### How to Run Unit tests?

The unit tests run when we are performing a maven build on the source, WSO2 has developed a maven-unittest-plugin inorder to run tests which is trigggered by a maven build. Running your unit tests is not straight forward when building using a CICD pipeline. This is because inorder to execute the unit tests maven needs a WSO2 MI/EI server present and accessible. When running the tests they will be deployed to the server and tested. In a nutshell there are two different ways to run unit tests.

1.  Using a local MI Server runtime

In this way the MI runtime should resides in the same machine we are building the source code. When executing the maven build command we need specify the path the

> mvn clean install -DtestServerType=local -DtestServerPath=/mnt/servers/wso2mi-1.2.0/bin/micro-integrator.sh

2\. Using a remote MI Server runtime.

In this way we can use a already running MI server runtime and we can specify the host/IP and the port of the remote server.

> mvn clean install -DtestServerType=remote -DtestServerHost=10.5.2.13 -DtestServerPort=9008

Now let’s look at how we can implementing a CICD Pipeline in a realworl scenario.

#### Method 01; Using a Remote MI Server

In this method, we are using a remote MI server to execute Unit tests, If you have a MI up and running already you can use this as well. But I will be setting up a MI runtime from the build pipeline itself. The easiest way to setup a MI runtime is to use a MI docker container. Hence I will be using docker to start a MI server and to implement the CICD pipeline I will be using Jenkins.

So let’s look at the implementation. The pipeline will first clone the integration source code from Github, then it will start an MI Container and then build the source with unit tests. The units tests will run on the MI container we are starting. Additionally we can deploy the artefacts to a lower environment and run integration tests on top of the integrations as well. The pipeline will simply look something similar to below.

![](/assets/img/medium/1__nFKTHkjYBYShjKBKQuXGzw.png)

The Jenkins pipeline code that is used given below. Please note that following code is for demo purposes and may not be production ready.

The full pipeline and the sample project can be found [here](https://github.com/yasassri/unittest-build-demo/blob/main/RemoteServer_Pipeline.yaml).

#### Method 02; Build your source within a Container

In this method we will be using a local MI server to run unitests. But we will not be setting up anything in the Jenkins agent nodes, instead we will setup a container with all the build dependencies and build the source within this container. Here we need a custom docker image with a few dependencies. Basically we need to setup maven in the Docker image so we can build the source within the Docker container. We will be using WSO2 MI base image to create the image. You can find the docker resources [here](https://github.com/yasassri/unittest-build-demo/tree/main/resources/docker).

Inorder to build the custom image follow the steps below.

1.  Clone the repo [https://github.com/yasassri/unittest-build-demo](https://github.com/yasassri/unittest-build-demo/tree/main/resources/docker)
2.  Navigate to [resources/docker](https://github.com/yasassri/unittest-build-demo/tree/main/resources/docker) and execute the following command.

docker build -t <IMAGE\_TAG\_NAME> .

I have already built the image and pushed it into the public docker registry. Hence you can use the image ycrnet/wso2mi-1.2.0-unittest:1.0.0 image as well.

After building the image and pushing it to a registry you can use the following pipeline to build the source with unit tests.

The pipeline will look like something below.

![](/assets/img/medium/1__kKBhNJEtVHkbvzTf7w3J6g.png)

Following is the source code of the Jenkins pipeline and the full pipeline source is available [here](https://github.com/yasassri/unittest-build-demo/blob/main/BuilInContainer_Pipeline.yaml).

That’s about it. Please drop a comment if you have any questions.