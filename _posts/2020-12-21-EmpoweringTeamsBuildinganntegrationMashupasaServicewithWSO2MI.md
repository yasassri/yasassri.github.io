---
title: >-
  Empowering your Teams; Building an Integration Mashup as a Service with WSO2
  MI
description: >-
  In medium-scale to large-scale corporations, there are numerous
  teams/business-units that operate cohesively yet independently to drive…
date: '2020-12-21T09:42:10.006Z'
categories: []
keywords: []
tags: [java, aws]
image:
  path: /assets/img/medium/0__Mgvcsevtq9ZbvROi.jpg
  width: 800
  height: 500
  alt: 
---

![](/assets/img/medium/0__BRQZDeO2CpEDqClN.jpg)

In medium-scale to large-scale corporations, there are numerous teams/business-units that operate cohesively yet independently to drive the organization towards its’ goals and objectives. These teams typically use different integral systems within their business units, which adds up to many different systems been adopted within the Organization. As the number of different systems grows within an Organization it’s going to be a nightmare to manage these different systems and technologies. On the other hand, it’s going to incur a considerable cost to the Organization. Hence as a proactive measure, it’s ideal for an organization to streamline and standardize the technology stack and the process, where the organization can build a common platform that can be used by the business units while eliminating the maintenance and management overhead. In simple terms, the Organization can develop a centralized SaaS platform where different business units can onboard themselves and start developing their integrations with a few clicks without worrying about the service management aspect. We will call this an “Integration Mashup as a Service(IMaaS)" where the platform will facilitate self-onboarding, service reuse, scalability, resilience, and above all streamline the integrations across the entire organization.

In this post, I will be exploring how WSO2 MI can be leveraged to develop an Integration Mashup as Service, Platform.

### Terminology

**Tenant**: A dedicated service space within the IMaaS platform that is separated from other business units where the relevant teams can build their services.

**Integrations**: Integrations include connecting systems, exposing your data as services, protocol switching to connect with different systems, exposing APIs, syncing different systems, payload transformations, processing files and updating systems, etc.

**Service**: An collection of integrations that will cater a business requirement.

### Designing the platform

#### Key Principles

Before diving into the design of the platform it’s crucial to identify the key principles that need to be considered when designing a SaaS platform. Hence the IMaaS platform should be developed keeping the following key principles in mind.

*   Scalability.
*   Performance.
*   Implementability.
*   High Availability.
*   Observability.
*   Data Privacy/Isolation.
*   Updatability.

Let’s look at each of the above principles in detail.

**Scalability**

Scalability is the ability of a system to handle a growing amount of work by adding resources to the system. Hence, the services should be able to scale depending on the load. This allows us to build a stable and fault-tolerant platform.

**Performance**

Performance expectancy is a key factor when designing any service platform. Hence the platform should be able to operate at an acceptable performance threshold.

**Implementability**

This refers to how easily the integrations can be built as microservices within the platform. This is a key factor in attracting new users to the platform.

**High Availability**

This captures the platform’s ability to maintain zero downtime. Which covers the aspects like failover management and the ability to perform rolling updates without shutting down the services.

**Observability**

One of the critical capabilities of any platform should be the ability to observe the services. Observability covers a few different aspects like, server resource monitoring, server matrics, message tracing, and log aggregating. Observability plays a key role when troubleshooting issues in the systems.

**Data Privacy/Isolation.**

This captures the platforms’ ability to isolate different tenant spaces and the ability to maintain data privacy. This requirement may differ from one organization to another.

**Updatability.**

This captures the platform's ability to update the services seamlessly and the ability to update the platform it-self easily.

#### High-level Design

The IMaaS will have multiple tenant spaces and each tenant space can run multiple services within the same space. This can be depicted as shown in the following diagram.

![](/assets/img/medium/1__IGl3r9fMDVOb__u__b69wpBg.png)

Honoring the key principles that were discussed in the previous section the platform can be designed to support the following capabilities. Hence a single tenant space will look something like below.

![](/assets/img/medium/1__XuZq5FrOpCPXOY1__YiHhJQ.png)

As shown in the above diagram, the services would run in a dedicated space and they can run as a single service or as clusters. At the same time, they can interact with other services. Overall the platform should support Log aggregation, message training, service matrics, service scaling, failover/rolling updates, and CICD.

Taking a step back and giving the platform a high-level look, at a very high-level the platform will look like something similar to the below diagram.

![](/assets/img/medium/1__mzVZXRypAB9isAVag1Lr3w.png)

As shown in the above image each tenant space should be isolated and each tenant space will have a dedicated control plane to mage the services. Also, an observability stack will be attached which allows developers to monitor the services and the platform. A state controller will make sure the services are rolled-out as per the developers' requirements and the states are maintained properly. The state controller will also make sure the services are scaled appropriately.

### Selecting the correct technology Stack

Now we have an idea of the key principles we need to follow when designing and developing the platform. Let’s see how we can select the ideal technologies to cater the core principles.

**Integration Platform**

The integration platform is the key component of the IMaaS platform which will let the tenant users build their services. So selecting and a proven, easy to adopt and container friendly tool is crucial in many ways. If the learning curve is too steep it will not be popular among the users. As the integration Platform, we will be using WSO2s’ MicroIntegrator which is an open-source, cloud-native integration framework with a graphical drag-and-drop integration flow designer and a configuration-based runtime for integrating APIs, services, data, and SaaS, proprietary, and legacy systems. Which makes it an ideal candidate for the IMaaS.

**Infrastructure**

Selecting an appropriate infrastructure is also crucial to make your platform reliable, resilient and manageable. One important thing to note is how much out of box features are available to full fill the platform requirements. For example, if rolling updates, failover, security, etc. are provided OOB from the infra layer it will take less effort to build up the platform. Hence given the micro nature of the services and segregation of the service spaces, the obvious option is Kubernetes. So as the underline infrastructure we could use an environment like K8S or any other variant of K8S like Openshift, EKS, etc.

**Workflow and Source Management**

In order to manage tenant onboarding, service onboarding, and service lifecycle management we will use a Gitops based approach, hence we will be using Github. We will also use Github to manage tenant-specific source code.

**CI and CD**

As the main CD tool, we can use Kubernetes native CD tools like ArgoCD or Spinnaker. I’m in favor of ArgoCD as we are following a Gitops based service lifecycle management process. Also as the CI tool, we can use a tool like Jenkins.

**Observability**

WSO2 MI supports different observability standards/tools out of the box which gives it an edge over other vendors available in the market. So as the observability stack we can use Grafana, Prometheus, Jaeger, Loki, and Fluent-Bit. Which allows monitoring the Server matrics, message tracing, and server log aggregation.

As a summary following are the tools we will be using for the solution we are building.

*   **K8S**: The underline infrastructure.
*   **WSO2 MicroIntegrator:** The integration platform.
*   **Jenkins**: CICD tool which takes care of the source building and overall deployment process.
*   **Spinnaker/ArgoCD**: CD tool used to roll-out/roll-back changes to K8S.
*   **Helm**: Configuration automation tool used for K8S artifacts.
*   **Github**: Gitops based operation management and source control.
*   **Grafana**: Centralized dashboard for tracing, metrics, and logs.
*   **Prometheus**: Used to capturing Matrics data.
*   **Loki and Fluent-bit**: Used to index the application logs.

### Reference Architecture

#### **Component Architecture**

Now let's look at a reference design we can create with the aforementioned toolset. At a glance, each tenant space would be something similar to the below component diagram. The following image shows where each component falls within the platform.

![](/assets/img/medium/1__9c00ntBi8Ot0NXzXbk2Q7Q.png)

In the above diagram, we have the WSO2 Micro-Integration stack, Observability stack, and other components that are needed to make the platform cater to the user requirements. We will talk about each flow in detail in the following sections.

#### Tenant and Service Onboarding and Lifecycle Management

We will be following a Git-ops based approach to onboard and manage tenant services. Hence the entry point for tenant onboarding will be a Git repo itself. Also, service onboarding and service lifecycle management will be done via Gitops. In order to adopt the Git-ops strategy, a repository structure can be designed as shown below.

![](/assets/img/medium/1__5Dc5NGKyGu7l152WAm5Adg.png)

As depicted in the above image, the platform will have a central repo where tenants can self onboard into the platform by creating a tenant metadata file. Then the tenants can onboard their services by adding service-related metadata to the tenants' metadata file. When a tenant-specific metadata file is committed to this repo, a separate namespace will be created in K8S for this particular tenant. Each Tenant repository will be pointing to multiple service repositories where it will have three branches representing each environment(Dev, Test, and Prod). The tenants should be able to omit the creation of environments if they wish to do so. Deployment of services to different environments will be handled by the commits happening in each of these branches. For example, the developers can keep on working on the dev/test branch, and when they are ready to release to the production they can simply merge the test branch with the production which will trigger a release pipeline.

Now let's look at how the tenant onboarding happens with the Gitops approach. The following image depicts the tenant onboarding flow.

![](/assets/img/medium/1__TM8W69__Gz8zRlWhrkOVC1A.png)

As shown in the above diagram initially a tenant user will send a PR to the IMaaS Repository which needs to be approved by platform admins. Once approved the changes will be picked up by a Jenkins Job and necessary resources for the tenant will be created. The process will create a dedicated namespace in K8S, a dedicated Docker registry in a private docker registry, and required CD pipelines in ArgoCD. Once the tenant is onboarded the tenants can start onboarding the services.

#### Service Lifecycle Management

This section explains how the service lifecycle can be managed. The following image depicts the CICD process of a tenant service. This process captures the flow that happens when the developers are moving the source from their local environments up to the development/test environment.

![](/assets/img/medium/1__hJKt__N9FjpXUA1qaY__GLeQ.png)

The above flow has two independent flows. The CI flow and the CD flow. The CI flow starts when the developers push the source code to the integration source repo or if they manually trigger the pipeline. First, the pipeline builds the integration source and runs unit tests on the source, after the source is built the deployment artifacts will be packed to a docker image and then the image is pushed to a Docker registry. After the updated docker image is pushed the CD process will be triggered by adding a commit to the tenant service repo. This will be picked by the CD job, the helm charts will be updated which will be picked by ArgoCD, and then the Environment will be updated with the latest changes.

Once the developers are confident that the development work is completed and the integrations are stable to be released to the production they can follow the below flow which captures production release process.

![](/assets/img/medium/1__2hvQP0M58kRRU0f5YEGmNA.png)

In the above flow in order to trigger the release process, the developers have to merge the staging/dev branch with the production branch. Merging into the production will trigger the production release.

That’s what I will be covering in this post. I’m planning to write a few follow-up articles to cover the implementation details of the proposed solution.

Feel free to drop a comment if you have any queries!!