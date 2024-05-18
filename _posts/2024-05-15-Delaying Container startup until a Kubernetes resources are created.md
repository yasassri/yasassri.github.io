---
title: Delaying application startup until Kubernetes resources are created
description: Delaying application startup until Kubernetes resources are created
date: '2024-05-15'
categories: [Kubernetes, WSO2]
keywords: [Kubernetes, WSO2, Clustering, Probes]
tags: [Kubernetes, WSO2, Clustering]
image:
  path: /assets/img/posts/containers.jpeg
  width: 800
  height: 500
  alt:
---
In this post, I will take you through the process I followed when trying to debug an issue that was occurring in one of the applications deployed in K8S. If you are just interested in the part mentioned in the title, skip to the second part of the blog.

# Finding what to fix

This was a unique problem I had to work on recently. It all started when WSO2 Enterprise Integrator, which is an ESB, started throwing the following exception. This application had been running for years without an issue, and suddenly the exception occurred. Weird!

```
ERROR {org.wso2.carbon.membership.scheme.kubernetes.KubernetesMembershipScheme} -  Kubernetes membership initialization failed
org.wso2.carbon.membership.scheme.kubernetes.exceptions.KubernetesMembershipSchemeException: No member ips found, unable to initialize the Kubernetes membership scheme
        at org.wso2.carbon.membership.scheme.kubernetes.KubernetesMembershipScheme.init(KubernetesMembershipScheme.java:98)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.initiateCustomMembershipScheme(HazelcastClusteringAgent.java:623)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.configureMembershipScheme(HazelcastClusteringAgent.java:604)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.createConfigForAxis2Mode(HazelcastClusteringAgent.java:402)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.loadHazelcastConfig(HazelcastClusteringAgent.java:448)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.init(HazelcastClusteringAgent.java:175)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.enableClustering(StartupFinalizerServiceComponent.java:293)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.completeInitialization(StartupFinalizerServiceComponent.java:187)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.serviceChanged(StartupFinalizerServiceComponent.java:317)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:451)
        at org.wso2.carbon.throttling.agent.internal.ThrottlingAgentServiceComponent.registerThrottlingAgent(ThrottlingAgentServiceComponent.java:123)
        at org.wso2.carbon.throttling.agent.internal.ThrottlingAgentServiceComponent.activate(ThrottlingAgentServiceComponent.java:100)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.eclipse.equinox.internal.ds.model.ServiceComponent.activate(ServiceComponent.java:260)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.activate(ServiceComponentProp.java:146)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.build(ServiceComponentProp.java:345)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponent(InstanceProcess.java:620)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponents(InstanceProcess.java:197)
        at org.eclipse.equinox.internal.ds.Resolver.getEligible(Resolver.java:343)
        at org.eclipse.equinox.internal.ds.SCRManager.serviceChanged(SCRManager.java:222)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:451)
        at org.wso2.carbon.core.init.CarbonServerManager.initializeCarbon(CarbonServerManager.java:515)
        at org.wso2.carbon.core.init.CarbonServerManager.start(CarbonServerManager.java:220)
        at org.wso2.carbon.core.internal.CarbonCoreServiceComponent.activate(CarbonCoreServiceComponent.java:105)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.eclipse.equinox.internal.ds.model.ServiceComponent.activate(ServiceComponent.java:260)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.activate(ServiceComponentProp.java:146)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.build(ServiceComponentProp.java:345)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponent(InstanceProcess.java:620)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponents(InstanceProcess.java:197)
        at org.eclipse.equinox.internal.ds.Resolver.getEligible(Resolver.java:343)
        at org.eclipse.equinox.internal.ds.SCRManager.serviceChanged(SCRManager.java:222)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.equinox.http.servlet.internal.Activator.registerHttpService(Activator.java:81)
        at org.eclipse.equinox.http.servlet.internal.Activator.addProxyServlet(Activator.java:60)
        at org.eclipse.equinox.http.servlet.internal.ProxyServlet.init(ProxyServlet.java:40)
        at org.wso2.carbon.tomcat.ext.servlet.DelegationServlet.init(DelegationServlet.java:38)
        at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1230)
        at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1174)
        at org.apache.catalina.core.StandardWrapper.load(StandardWrapper.java:1066)
        at org.apache.catalina.core.StandardContext.loadOnStartup(StandardContext.java:5433)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5731)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:145)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1707)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1697)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
[localhost-startStop-1] ERROR {org.wso2.carbon.core.internal.StartupFinalizerServiceComponent} -  Cannot initialize cluster
org.apache.axis2.clustering.ClusteringFault: Kubernetes membership initialization failed
        at org.wso2.carbon.membership.scheme.kubernetes.KubernetesMembershipScheme.init(KubernetesMembershipScheme.java:111)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.initiateCustomMembershipScheme(HazelcastClusteringAgent.java:623)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.configureMembershipScheme(HazelcastClusteringAgent.java:604)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.createConfigForAxis2Mode(HazelcastClusteringAgent.java:402)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.loadHazelcastConfig(HazelcastClusteringAgent.java:448)
        at org.wso2.carbon.core.clustering.hazelcast.HazelcastClusteringAgent.init(HazelcastClusteringAgent.java:175)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.enableClustering(StartupFinalizerServiceComponent.java:293)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.completeInitialization(StartupFinalizerServiceComponent.java:187)
        at org.wso2.carbon.core.internal.StartupFinalizerServiceComponent.serviceChanged(StartupFinalizerServiceComponent.java:317)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:451)
        at org.wso2.carbon.throttling.agent.internal.ThrottlingAgentServiceComponent.registerThrottlingAgent(ThrottlingAgentServiceComponent.java:123)
        at org.wso2.carbon.throttling.agent.internal.ThrottlingAgentServiceComponent.activate(ThrottlingAgentServiceComponent.java:100)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.eclipse.equinox.internal.ds.model.ServiceComponent.activate(ServiceComponent.java:260)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.activate(ServiceComponentProp.java:146)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.build(ServiceComponentProp.java:345)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponent(InstanceProcess.java:620)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponents(InstanceProcess.java:197)
        at org.eclipse.equinox.internal.ds.Resolver.getEligible(Resolver.java:343)
        at org.eclipse.equinox.internal.ds.SCRManager.serviceChanged(SCRManager.java:222)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:451)
        at org.wso2.carbon.core.init.CarbonServerManager.initializeCarbon(CarbonServerManager.java:515)
        at org.wso2.carbon.core.init.CarbonServerManager.start(CarbonServerManager.java:220)
        at org.wso2.carbon.core.internal.CarbonCoreServiceComponent.activate(CarbonCoreServiceComponent.java:105)
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
        at java.lang.reflect.Method.invoke(Method.java:498)
        at org.eclipse.equinox.internal.ds.model.ServiceComponent.activate(ServiceComponent.java:260)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.activate(ServiceComponentProp.java:146)
        at org.eclipse.equinox.internal.ds.model.ServiceComponentProp.build(ServiceComponentProp.java:345)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponent(InstanceProcess.java:620)
        at org.eclipse.equinox.internal.ds.InstanceProcess.buildComponents(InstanceProcess.java:197)
        at org.eclipse.equinox.internal.ds.Resolver.getEligible(Resolver.java:343)
        at org.eclipse.equinox.internal.ds.SCRManager.serviceChanged(SCRManager.java:222)
        at org.eclipse.osgi.internal.serviceregistry.FilteredServiceListener.serviceChanged(FilteredServiceListener.java:107)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.dispatchEvent(BundleContextImpl.java:861)
        at org.eclipse.osgi.framework.eventmgr.EventManager.dispatchEvent(EventManager.java:230)
        at org.eclipse.osgi.framework.eventmgr.ListenerQueue.dispatchEventSynchronous(ListenerQueue.java:148)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEventPrivileged(ServiceRegistry.java:819)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.publishServiceEvent(ServiceRegistry.java:771)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistrationImpl.register(ServiceRegistrationImpl.java:130)
        at org.eclipse.osgi.internal.serviceregistry.ServiceRegistry.registerService(ServiceRegistry.java:214)
        at org.eclipse.osgi.framework.internal.core.BundleContextImpl.registerService(BundleContextImpl.java:433)
        at org.eclipse.equinox.http.servlet.internal.Activator.registerHttpService(Activator.java:81)
        at org.eclipse.equinox.http.servlet.internal.Activator.addProxyServlet(Activator.java:60)
        at org.eclipse.equinox.http.servlet.internal.ProxyServlet.init(ProxyServlet.java:40)
        at org.wso2.carbon.tomcat.ext.servlet.DelegationServlet.init(DelegationServlet.java:38)
        at org.apache.catalina.core.StandardWrapper.initServlet(StandardWrapper.java:1230)
        at org.apache.catalina.core.StandardWrapper.loadServlet(StandardWrapper.java:1174)
        at org.apache.catalina.core.StandardWrapper.load(StandardWrapper.java:1066)
        at org.apache.catalina.core.StandardContext.loadOnStartup(StandardContext.java:5433)
        at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5731)
        at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:145)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1707)
        at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1697)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
Caused by: org.wso2.carbon.membership.scheme.kubernetes.exceptions.KubernetesMembershipSchemeException: No member ips found, unable to initialize the Kubernetes membership scheme
        at org.wso2.carbon.membership.scheme.kubernetes.KubernetesMembershipScheme.init(KubernetesMembershipScheme.java:98)
        ... 80 more
```

The thought process was that it had been working fine for three years and suddenly broke, so obviously, the Ops team must have made some changes to the K8S cluster, right? :) After checking with them, it turned out they hadn't made any changes recently. So, we had to find out what was going on. Let's look at the exception closely: it's thrown from the clustering component of WSO2. When you want tasks to be coordinated in a multinode WSO2 application setup, you can enable clustering in WSO2. For clustering to work, WSO2 should be able to discover the application nodes and communicate with each other. WSO2 clustering uses Hazelcast underneath with a custom membership schema (this is where you tell Hazelcast how to discover other members of the cluster).

So, let's put on the developer hat. According to the stack trace, the exception is thrown from the `org.wso2.carbon.membership.scheme.kubernetes.KubernetesMembershipScheme` class. Since WSO2 is an open-source product, we can look into the code and see what exactly is happening. I won't go into details, but basically, WSO2 is calling K8S APIs to retrieve member details. If anyone is interested, here is where the exception is thrown: https://github.com/wso2/kubernetes-common/blob/v1.0.4/kubernetes-membership-scheme/src/main/java/org/wso2/carbon/membership/scheme/kubernetes/KubernetesMembershipScheme.java#L97.

Let's see how clustering works in WSO2, so we can better understand the issue. Basically, a pod is exposed via a K8S service. When a service is created, K8S will create endpoints to represent each pod, which are grouped based on the selectors. So, in order to identify the members and their IPs, WSO2 calls the Endpoints API in K8S (https://kubernetes.io/docs/reference/kubernetes-api/service-resources/endpoints-v1/#http-request), which returns a list of endpoints registered and their states. The response will be something like below.

```
 Name: "mysvc",
 Subsets: [
   {
     Addresses: [{"ip": "10.10.1.1"}, {"ip": "10.10.2.2"}],
     Ports: [{"name": "a", "port": 8675}, {"name": "b", "port": 309}]
   },
   {
     Addresses: [{"ip": "10.10.3.3"}],
     Ports: [{"name": "a", "port": 93}, {"name": "b", "port": 76}]
   },
]
```

From the above response, WSO2 will get all the endpoint IPs and register them as cluster members. However, for some reason, the endpoints are not being returned by the API call.

Long story short, after debugging, this is what was happening. For K8S to register endpoints, the pod should be in the `Running` state. For that to happen, all the containers within the pod should be successfully pulled and started. What was happening is that we had a Fluentbit sidecar container associated with the main container for logging. For some reason, the Fluentbit image was taking more time than the main container to be pulled. So, the main container gets pulled and starts, but the sidecar takes more time to get pulled. Hence, when WSO2 calls the K8S API to get the endpoints, they are not yet registered because the pod is not in the Running state. Later on, we learned that a new corporate proxy was added, which caused the slowness in the image pull process.

Ideally, WSO2 should have added some logic to retry if the endpoints were not registered, but they haven't, so we decided to implement a workaround for the issue. Now let's get to the solution, the title of the blog :)

# Delaying Application Startup

Let's see how we can delay the application startup until a certain condition is met. In this case, I will check if the K8S endpoints are registered before starting the application. There are two issues associated with this. First, we need to delay the application startup. Then, we need to make sure health check probes are not failing because the waiting time will be dynamic.

## Delaying the App Startup

To do this, we can simply add some logic to the init script. For example, something like below:

```sh
#!/bin/bash
echo "Starting the entrypoint"
# Endpoint API URL
KUBE_API_URL=https://${KUBERNETES_SERVICE_HOST}:443/api/v1/namespaces/${KUBERNETES_NAMESPACE}/endpoints/${KUBERNETES_SERVICES}
# Bearer token file path
BEARER_TOKEN_FILE="/var/run/secrets/kubernetes.io/serviceaccount/token"
# Number of retry attempts.
RETRY_COUNT=24
# Sleep time between retries (in seconds)
RETRY_INTERVAL=5
# Function to read Bearer token from file
get_bearer_token() {
    if [ -f "$BEARER_TOKEN_FILE" ]; then
        cat "$BEARER_TOKEN_FILE"
    else
        echo "Error: Bearer token file not found at $BEARER_TOKEN_FILE"
        exit 1
    fi
}
# Function to make the API call and check for "Subsets" in the JSON response
# This is needed for EI clustering.
check_api() {
    bearer_token=$(get_bearer_token)
    response=$(wget --no-check-certificate --header="Authorization: Bearer $bearer_token" -qO- $KUBE_API_URL)
    if [[ $response == *"notReadyAddresses"* || $response == *"addresses"* ]]; then
        echo "Subset section found in the response!"
        return 0
    else
        echo "Subset section not found. Retrying..."
        return 1
    fi
}

echo "Checking the endpoint API ${KUBE_API_URL}"
# Retry loop
attempt=1
while [ $attempt -le $RETRY_COUNT ]; do
    check_api
    exit_code=$?
    if [ $exit_code -eq 0 ]; then
        # Adding a flag so we can use this for the startup probe
        touch /home/wso2carbon/check-done  
        # Cleaning the history
        history -c
        # Exec wso2 entrypoint
        echo "Starting the application"
        exec /home/wso2carbon/docker-entrypoint.sh
    fi
    echo "API call failed attempt: ${attempt} and sleeping for ${RETRY_INTERVAL}"
    sleep $RETRY_INTERVAL
    ((attempt++))
done

# If we reach here, all retrty attempts failed.
echo "Failed after $RETRY_COUNT attempts. Subset section not found."
exit 1

```
As you can see in the script, I will be calling the Endpoints API to check if the endpoints have been registered: https://${KUBERNETES_SERVICE_HOST}:443/api/v1/namespaces/${KUBERNETES_NAMESPACE}/endpoints/${KUBERNETES_SERVICES}. Also, note that I have already created a service account with the necessary permissions to call the K8S API, and the token for this is mounted to `/var/run/secrets/kubernetes.io/serviceaccount/token`. As soon as the condition is met, we will create a file indicating the process was completed and start the application.

## Adjusting the Probes

Since the waiting time is dynamic, we have to make sure the `readinessProbe` is not started until the application is fully started. Otherwise, as soon as the container starts running, the `readinessProbe` will kick in and fail after the defined retries. Adding an initial delay is not a good solution as the wait time can vary. So as a solution, we can add a `startupProbe`. The `startupProbe` ensures the other health probes are not triggered until the startupProbe is successful. If you look at the above script, we are creating a file named /home/wso2carbon/check-done when the waiting is completed. In the `startupProbe`, you can simply check the availability of this file. The probes will look like below.

 ```yaml
 startupProbe:
  initialDelaySeconds: 15
  periodSeconds: 5
  failureThreshold: 36
  successThreshold: 1
  exec:
    command:
    - /bin/sh
    - -c
    - test -f /home/wso2carbon/check-done
readinessProbe:
  initialDelaySeconds: 20
  periodSeconds: 15
  failureThreshold: 1
  successThreshold: 1
  tcpSocket:
    port: {{ .Values.deployment.ports.gw.containerport }}
 ```

*Note:* _If you are using this solution for WSO2 applications, the WSO2 app will create a process ID file when the application starts, so you don't have to create additional files. You can simply check the availability of the /home/wso2carbon/<PRODUCT_HOME>/wso2carbon.pid file._

So that's basically it. I hope this will be helpful to someone. Drop a comment if you have any questions.