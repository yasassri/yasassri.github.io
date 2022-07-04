---
title: Perfect Your Ballerina Act — With Testerina
description: 'Designing a Testable Application'
date: '2018-05-01T04:24:02.933Z'
categories: [WSO2, Ballerina]
tags: [ballerina, wso2]
image:
  path: /assets/img/posts/ballerina.jpg
  width: 800
  height: 500
  alt: 
---
### Designing a Testable Application

This post is not about dancing!! Rather it’s about developing a microservice based **testable** end-to-end application with Ballerina and making sure it runs perfectly without any glitches using Testerina.

Let me explain the unfamiliar terms! What is [**Ballerina**](https://ballerina.io/)? Ballerina is a general purpose, concurrent, strongly typed and cloud native programming language which focuses on integrating and orchestrating microservices etc. **Testerina** is the native testing framework built into Ballerina which enables Ballerina developers to capitalize on test driven development. It’s similar to TestNG for JAVA, Go-test for Go-lang etc.

### Along the Test Pyramid,

Before we go into details let’s look at some well-known testing strategies. The test pyramid is one popular strategy we can adopt when designing tests. Test pyramid illustrates the composition of tests which is ideal to have at different testing layers. Following image gives an idea about the test pyramid.

![](/assets/img/medium/1__XUXMGnPlTmvcqrlkZzvFUw.png)

It’s notable that when you move up the pyramid the execution time for tests and the component size of the application may increase. So as a rule of thumb it’s ideal to have more unit tests and less end to end tests. It doesn’t mean you shouldn’t have end-to-end tests at all, we should find the correct mix of tests ideal for your application. Lets dive a little deeper and understand each layer better.

#### Unit Testing.

An unit represents an isolated code fragment that can be tested without any external dependencies. For example, A function which adds two numbers and return the sum.

#### Integration Testing.

Integration tests are done when multiple components integrate with each other. For e.g: Persistence layer interacting with the DB, Service layer interacting with the service implementations etc.

#### End to End Testing.

End to end testing layer depicts, testing a production equivalent deployment of the entire solution. The end to end testing deployment will mimic the actual production deployment and perform tests on top of this environment.

### Design decisions; Making your application \*Testable\*

In-order to efficiently test your application it should be designed in a testable manner. This involves a set of conscious design decision a developer should take to make sure the application is testable, not as a monolithic app but as a set of units which aligns with the test pyramid strategy. And the approach you follow to make your application testable may vary from application to application. In this post, I will use a sample based on microservices, to explain how you can design a testable application with Ballerina and test it with Testerina.

### Theatre; A stage to demonstrate

Lets come up with a sample application. Theatre Management App; An event and ticket management application. This will explain how an end to end application can be designed based on microservices.

I have modeled the sample application as a collection of microservices which orchestrates the functional flow of the use-cases. Theatre app contains 3 microservices (Portal, TicketingMS, EventHandlerMS). Following image depicts the high level architecture of the sample. Please note that Payment Gateway is an external service which only acts as an Endpoint.

![](/assets/img/medium/0__A85khTKGc50Hd__rv.png)

#### Portal (Composite Application)

Portal acts as the composite service which does the plumbing between other microservices and the external service. (Ticketing, Event and Payment Gateway). Only the portal app will be exposed as the public user interface and it will route and mediate the incoming messages.

#### Event Handler MS

This service is responsible for handling events, registering events, retrieving event information etc.

#### Ticketing MS

Ticketing service is responsible for handling ticketing related operations. Viewing available Tickets, purchasing tickets etc.

### Theatre; Message Flows

Let’s look at the main message flows of the Theatre application. Mainly the Event organizers can register their event with available ticket information. Then the users can browse through the events and purchase tickets by providing credit card information.

#### Register Event Message flow

Following sequence diagram depicts the event registration message flow. The user will be sending event and ticket information in a single request which will be extracted by portal service and sent to other microservices.

![](/assets/img/medium/0__z56dxaodq3C8ZYtp.png)

As you can see, the register message is routed through different services and the flow is orchestrated by the Portal service.

#### Purchase Ticket Message flow

Once an event is added, the users can browse the events and purchase the tickets. Following sequence diagram depicts the ticket purchasing message flow.

![](/assets/img/medium/0__gsrATzqqPyGZvtGF.png)

### Breaking the monolithicity

If we look at the L0 architecture of the solution, It’s quite large to be tested as a whole. It interacts with external services which you may not have any control over. And also keep in mind the Test Pyramid, you should always minimize testing at end-to-end layer.

We can break the entire solution into following testable isolated components.

![](/assets/img/medium/0__Lc2Sz0GbWBzB7zv__.png)

Each separated component can be tested isolated, by mocking the interactions with each other.

### Microservice Architecture

If we further dive into the actual microservice, I have componetise the microservices which will allow me to separate and isolate functionality. Following is the microservices component architecture each service is made of. This architecture may vary from application to application.

![](/assets/img/medium/0__aXHsTOuCTgh3KMzm.png)

*   **Services/Resources :** The actual service interfaces exposed to the users, where requests are accepted and taken in.
*   **Service Implementations :** Service layer dispatches the request information to service Implementation layer where the payload get processed.
*   **Models :** Ballerina objects to store your data. e.g : Register Event Request is represented as an object, when the request arrives, an object (A Model) is created with request information, which is parsed around the program.
*   **Utility Functions** : Functions that act as helper functions, used for different operations. e.g : Generate a json error from a string etc.
*   **Persistence** : Persistence functions takes care of DB interactions.
*   **Connectors** : Connectors are responsible for calling external services and other micro services.

Messages flow through a microservice as illustrated in the following diagram.

![](/assets/img/medium/1__P5FBjW7yT5ocYskUQ0kHMg.png)

As you can see above, the message gets strip down and validated along the execution path. The response path is also similar to the above.

### Identifying the testable units

So lets see how we can identify testable components from the sample app. If we look into individual microservices closely, following is how a microservice can be further dissected into testable components. The basis for selecting components is the functional independence of each component.

![](/assets/img/medium/0__AnmLjBi91SD9X4kM.png)

Let me explain the reasons behind drawing the above boundaries around different components.

**Models Test Boundary**

Models are the smallest entities that can be tested, and the accessibility of model elements can be tested here so It’s a self contained testable component.

**Utility Functions Test Boundary**

Utility functions can also be tested isolated, since most of them do not depend on any other functions. At this level we can test the individual utility functions extensively with less distractions.

**Service Impl Testing Boundary**

Then if we consider service implementation boundary, It contains models, utility functions and the service implementation which needs to be tested. We can try to isolate the service implementation layer from models and utility functions but it will not give an advantage, since mocking the utility functions may needs more effort than using the actual function it self. Another thing to note is the best way to test your function is without mocking interactions since it reflects the actual interaction then. But again the

But the persistence layer should be decoupled from the service impl layer since it requires a database to connect, hence we should introduce a mocking layer between the service implementation layer and persistence layer.

**Service Testing boundary**

Service testing boundary includes Service Imple, utility functions and models. Since interaction between these components are not heavy, we can include the actual components without mocking them. Again we need to mock the persistence layer and the connectors to test the service layer.

**Connectors test boundary**

Connectors also can be tested isolated by mocking the external interactions.

**Persistence test boundary**

We can test the persistence layer by mocking the Database interactions.

So that wraps up this post. Please drop a comment if you have thought or queries. Next post will explain how the actual app looks like and how we can write tests for our application with Testerina.