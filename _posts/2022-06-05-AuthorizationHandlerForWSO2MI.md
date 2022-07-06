---
title: Role Based Authorization Hanlder for WSO2 Micro Integrator
description: Authentication and Authorization handler for WSO2 MI
date: '2022-06-05'
categories: [WSO2, WSO2MI]
keywords: []
tags: [wso2, wso2mi, java, authorization]
image:
  path: /assets/img/posts/padlock.jpg
  width: 800
  height: 500
  alt:
---

WSO2 Micro Integrator(MI) is a lightweight integration platform that allows you to write different integrations to connect systems just like an Enterprise Service Bus. If you are familiar with the WSO2 products stack, WSO2 MI is the descendent of Enterprise Integrator and MI will be eventually replacing EI.

When creating different services that serve your enterprise traffic it's important to keep your services secure. For example, once you create an API or a Proxy service in WSO2 MI you should be able to enforce Authentication and Authorization. Authentication will determine whether the user trying to access the service is valid or whether the user is registered in the system and Authorization will determine whether the registered user has privileges to access a service. In other terms, authentication validates your credentials and authorization will check whether you have permissions assigned to you. Normally the permissions are determined y the roles that are assigned to a specific user. 

With the default product, there is no way to enforce authorization for an API/Proxy , hence I came up with a custom Authorization handler.  The next section will explain how you can use the authorization handler. 

### Custom Authorization Handler ###
The handler is capable of handling authentication and authorization separately. The user can disable authorization and only use authentication if necessary. Following is a simple flow diagram that illustrates the logical flow. 

![](/assets/img/posts/authFlow.jpeg){: w="300" h="500" }

As you can see above if the handler is engaged in a service, it will do authentication first and if authorization is enabled the authorization will be checked. When adding authorization the users can define a list of roles that are allowed to access the service. 

### How to use the Authorization Handler ###

1. First build this project or download the released Jar from https://github.com/yasassri/wso2mi-authorization-handler/releases/tag/v1.0.0 and copy the `wso2-authorization-handler-*.jar` to `<MI_HOME>/lib` directory.

*Note: Make sure you download the latest release.* 
 
2. Then in your API/Proxy service definition you can add the handler as shown below.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<api xmlns="http://ws.apache.org/ns/synapse" name="test2" context="/test2" binds-to="default">
   <resource methods="POST" binds-to="default">
      <inSequence>
         <payloadFactory media-type="xml">
            <format>
               <response>Hello</response>
            </format>
            <args/>
         </payloadFactory>
         <respond/>
      </inSequence>
      <outSequence/>
      <faultSequence/>
   </resource>
   <handlers>
    <handler class="com.ycr.auth.handlers.AuthorizationHandler">
      <property name="roles" value="admin,test" />
      <property name="authorize" value="true" />
    </handler>
</handlers>
</api>
```

3. Then add the user credentials as a `Basic Auth` header to your request and send the request. Refer following. (Make sure user:password is base64 encoded)

```sh
curl -v -X POST http://localhost:8290/test2 -H "Authorization: Basic YWRtaW46YWRtaW4="
```
*Note: If the Authentication fails you will get a HTTP 401 or if the Authorization fails you will receive a HTTP 403.*

**Handler params.**

Handler accepts two parameters `roles` and `authorize`. 

```xml
<handler class="com.ycr.auth.handlers.AuthorizationHandler">
      <property name="roles" value="admin,test" />
      <property name="authorize" value="true" />
</handler>
```

- **roles**: The user can define a list of allowed roles for the API.
- **authorize**: If Authorization(Role validation) is not required this can be set to false. If set to false only authentication will take place. The authorization stage will be skipped. 

*Note: In order to do role management you need to plugin an LDAP or a JDBC user store to MI.* 

Hope this post helps. Happy Coding! 
