---
title: Exposing Openshift Route with a Loadbalancer.
description: >-
  Openshift is the commercial Kubernetes offering that is built and backed by
  RedHat. Although the underline runtime of Openshift is…
date: '2020-03-16T16:53:06.319Z'
categories: [Devops, Openshift]
keywords: []
tags: [loadbalancing, openshift, nginx, haproxy, tcp]
image:
  path: /assets/img/medium/0__Ha5OaVmAjHpMaHWM.jpg
  width: 800
  height: 500
  alt: Amazon AWS
---

Openshift is the commercial Kubernetes offering that is built and backed by RedHat. Although the underline runtime of Openshift is Kubernetes there are few differences when it comes to Openshift from Kubernetes. Mainly traffic routing, scaling/rollbacks, and deployments are different in Openshift. I’m not going to talk much about Openshift or Kubernetes in this post. In this post, I will explain how we can front an Openshift Route with an external load balancer.

Route is the mechanism that allows you to expose an Openshift service externally, this is similar to an Ingress in Kubernetes. By default, a route in Openshift is an HA Proxy. When you create a route it will expose your services through an HA Proxy router. There is a Controler for the router which dynamically updates the router configurations as it detects changes. But the pluggable architecture of Openshift routes allows you to change this to any other implementation you like and it supports plugins like F5. You can read more information about this from the Openshift website itself.

Before we start the configurations we need to understand how the actual routing happens. When a request is received to the Openshift router the router will use the SNI(Server Name Identifier) information in the HTTP handshake to identify the service the request should be routed to. For example, if there are multiple services that are running in Openshift, the service calls are differentiated with the SNI information. You can refer to the following as an example.

![](/assets/img/medium/0__dW4YjlBXqqTHNwRH.jpg)

In this post, we look at how we can front the Openshift route with an external load balancer. There are many occasions you will have to do this. In most cases, within an organization, the internal applications will be exposed to the internet through a Loadbalancer to isolate the networks. In my case, I had two different Openshift clusters (DR and Production) which I had to loadbalance between, So I had to front these two Openshift clusters with an F5 load balancer. In this post, I will not use F5, but I will be using Nginx as an application Loadbalancer and HAProxy as a TCP loadbalancer. Following is how the end-result will look.

![](/assets/img/medium/0__Gt3NoMD3dr7jgWjy.jpg)

From the above diagram, it seems like a simple thing to do but it isn’t. As you can see I have two hostnames to access the services, one hostname for the internal users and one for the external users. Since Openshift routes rely on SNI information in the handshake to decide the service the request should be sent it’s not straight forward to achieve this.

There are multiple ways to achieve this, I will explain each method in the below section.

1.  **Using a TCP Loadbalancer**

The easiest way to achieve this is using a TCP loablancer to stream the requests directly to the route, but what’s important is in this case you need to create two different routes, one for the public DNS and the other with the internal Openshift DNS. Traffic thats originating externally will reach the route with the public DNS and the others to the internal route.

![](/assets/img/medium/0____7miiFsUwbbX__gKS.jpg)

I have used HAProxy to demonstrate this. Following is the haproxy.cfg I used to achieve this.

```yaml
global
    log 127.0.0.1   local0
    log 127.0.0.1   local1 debug
    maxconn 4096

  defaults
    log     global
    mode    http
    option  httplog
    option  dontlognull
    retries 3
    option redispatch
    maxconn 2000
    timeout connect      5000
    timeout client      50000
    timeout server      50000

  frontend is.wso2.com
    bind *:443
    mode tcp
    default_backend nodes

  backend nodes
    mode tcp
    balance roundrobin
    server server1 192.168.64.7:443 check
```

**2\. Using an Application Loadbalancer.**

This is the more complex approach where you will terminate the connection at the main LB and initiate a new connection with the Openshift Route. As I mentioned earlier Openshift route identifies the appropriate route with the SNI information. So I had to set the SNI information when the second connection is made from the External Loadbalancer to Openshift Route.

![](/assets/img/medium/0__oIiVQ0eH9__2TBtnh.jpg)

Following is the NginX configuration that you can use to achieve this. Please note the important properties like **proxy\_ssl\_name, proxy\_ssl\_server\_name** which are used to set new SNI information for the connection.

```json
server {
  listen 443 ssl;
  server_name employee.external.com;
  ssl_certificate /usr/local/etc/nginx/keys/wso2.crt;
  ssl_certificate_key /usr/local/etc/nginx/keys/wso2.key;
  location / {
    proxy_set_header Host employee.apps-crc.testing;
    proxy_pass https://192.168.64.7;
    proxy_redirect https://employee.apps-crc.testing/ https://employee.external.com/;
    proxy_ssl_name employee.apps-crc.testing;
    proxy_ssl_server_name on;
    }
}
view raw

```

In the above solution, you can use the single route for exposing the services externally and internally. But I would recommend using solution 1 for the above problem to make things simple.

Please drop a comment if you have any questions.