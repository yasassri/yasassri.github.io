---
title: Testerina — Power of Mocking Revealed
description: >-
  Testerina is the built-in test framework for Ballerina. If anyone is wondering
  what is ballerina? Ballerina is a general purpose…
date: '2018-06-04T09:09:31.568Z'
categories: [WSO2, Ballerina]
tags: [ballerina, wso2, testing]
image:
  path: /assets/img/medium/0__q9dEl9CqTReyVSVp.jpg
  width: 800
  height: 500
  alt: 
---
Testerina is the built-in test framework for Ballerina. If anyone is wondering what is ballerina? Ballerina is a general purpose, concurrent, strongly typed and cloud native programming language which focuses on integrating and orchestrating microservices etc. If you want to learn more about ballerina you should read the rich content available at [ballerina.io](https://ballerina.io/)

This post will explain the mocking capabilities testerina provides, which allows you to write independent self-contained tests for your application.

### Why mock?

Why should we mock in the first place? Following are some cases where mocking is required.

1.  **Too many integration points.**

A decade ago applications were linear, small, self contained and isolated. We used to have standalone applications which ran by it self, but not anymore. Todays applications are distributed and connected with many other integral components and endpoints. So when it comes to testing these connected application it’s not practically possible to test with all the real connections. So we need to mock these endpoints when we are testing these applications.

**2. Demand for higher rate of delivery.**

In good old days we had SDLC models like waterfall model, which is a lengthy process and releases only happened after months of development and testing. But nowadays we have daily releases, with CICD coming into the picture, build time of an application does matter a-lot when an application is to be delivered in quick pace. If the build takes days this would affect the release cycles and developers efficiency. So if we are running our tests with real component integrations which sometimes consumes more time (e.g : database integration) the release process will be drags to test execution time. So in-order to cutdown the test execution time we can mock the components that consumes more time.

**3. Test driven development**

Test driven development is one of the most popular trends in software development today. In test driven development developers writeup their tests first and keep on developing the feature till all the tests are passing. So what if the component you are building need to interact with a different component which is developed by a different team, can you wait for that feature to be completed? So again we might have to mock that component and do the development.

Having explained why mocking is needed, lets see how testerina facilitates mocking. Testerina has two types of mocking,

1.  Function mocking.
2.  Service mocking.

### Function mocking

Testerina provides the functionality to mock a function in a different third-party package with your own Ballerina function which will help you to test your package independently. Following is a code sample which uses function mocks.

This feature is very powerful and can even be used to mock inbuilt functions, but this should be used with care. Following is an example where I have mocked the inbuilt println function.

### Service Mocking

#### Create a mock service with ballerina service capabilities

Testerina allows you to use existing ballerina service capabilities and start a mock services when a test requires a backend mock service. Following example shows how this can be done.

In the above sample, for our tests to work we are starting a mock service called _HelloServiceMock._ Service startup is triggered in the before test function with startServices function

#### Create a service mock with a swagger definition

If you have a swagger definition, you can use that to create a mock service for your tests. Following is how you can generate a mock service out from a swagger definition.

In the above sample in the init function we are calling the startServiceSkelaton method with the yaml definition which will start the service with the given name.

This wraps-up this post, please feel free to drop a comment if you have any queries.