---
title: Restricting Access to WSO2 Servers
description: >-
  In this post I’ll explain how we can restrict access to WSO2 Carbon console
  for specific client machines. In WSO2 servers they have an…
date: '2019-06-29T06:23:28.794Z'
categories: []
keywords: []
slug: /@ycrnet/restricting-access-to-wso2-servers-a6b063251a3c
---

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__1VoHbE5NJchauQxn.jpg)

In this post I’ll explain how we can restrict access to WSO2 Carbon console for specific client machines. In WSO2 servers they have an embedded tomcat instance to host the web-apps including the server. So here we will be using tomcat valves to restrict access to the server. I’ll be covering few different scenarios where you will have to use different configurations and valves.

#### White List a IP When Accessing the Server Directly

Take a scenario where the WSO2 server is accessed directly without a load balancer. So in this case how can we whitelist only requests generating from a particular IP or block requests originating from a particular IP.

In-order to do this we need to use tomcats’ RemoteAddrValve. Follow the steps below to add the valve.

1.  Open <WSO2\_HOME>/repository/conf/tomcat/carbon/META-INF/context.xml and add the valve configurations as below,

<Valve className=”org.apache.catalina.valves.RemoteAddrValve” allow=”10\\.100\\.5\\.112"/>

2\. Then restart the server for changes to be effective.

Above filter will only allow requests that are originating from the 10\\.100\\.5\\.112 IP. Also note the value for the allow property accepts a regex, so you can add a proper regex pattern here. There are few attributes you can set in the valve. You can also add multiple IPs, deny access to specific IPs etc. You can refer the tomcat documents [https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Remote\_Address\_Filter](https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Remote_Address_Filter) for more details on this.

#### White List a Host Name When Accessing the Server

In this case we are validating whether the requests are generated for a specific Host. By default Tomcat doesn’t do a DNS lookup on the clients IP to determine the Host name if the client. So we need to enable this in the relevent tomcat connector. After enabling dns lookup you have to use RemoteHostValve in tomcat to validate the host. Follow instructions below to do this.

1.  Open To do that open “_<IS\_HOME>/repository/conf/tomcat/catalina-server.xml”_ and enable _enableLookups_ property. Full connector configurations will look like following.

<Connector protocol=”org.apache.coyote.http11.Http11NioProtocol”  
 port=”9443"  
 bindOnInit=”false”  
 sslProtocol=”TLS”  
 sslEnabledProtocols=”TLSv1,TLSv1.1,TLSv1.2"  
 maxHttpHeaderSize=”8192"  
 acceptorThreadCount=”2"  
 maxThreads=”250"  
 minSpareThreads=”50"  
 disableUploadTimeout=”false”  
 **enableLookups=”true”**  
 connectionUploadTimeout=”120000"  
 maxKeepAliveRequests=”200"  
 acceptCount=”200"  
 server=”WSO2 Carbon Server”  
 clientAuth=”want”  
 compression=”on”  
 scheme=”https”  
 secure=”true”  
 SSLEnabled=”true”  
 compressionMinSize=”2048"  
 noCompressionUserAgents=”gozilla, traviata”  
 compressableMimeType=”text/html,text/javascript,application/x-javascript,application/javascript,application/xml,text/css,application/xslt+xml,text/xsl,image/gif,image/jpg,image/jpeg”  
keystoreFile=”${carbon.home}/repository/resources/security/wso2carbon.jks”  
keystorePass=”wso2carbon”  
URIEncoding=”UTF-8"/>

2\. Next open <WSO2\_HOME>/repository/conf/tomcat/carbon/META-INF/context.xml and add the following valve configuration,

<Valve className=”org.apache.catalina.valves.RemoteHostValve” allow=”wso2\\.mydomain\\.com"/>

3\. Then restart the server for changes to get effective.

Now if you try to access the WSO2 server with [https://wso2.mydomain.com](https://wso2%5C.mydomain%5C.com):9443/carbon you will be able to access but if you try to access the server with a different host you will be access denied. You can get more details on this valve from here [https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Remote\_Host\_Filter](https://tomcat.apache.org/tomcat-7.0-doc/config/valve.html#Remote_Host_Filter)

#### White List a Client IP When Accessing the Server through a LB

When you are accessing WSO2 servers through a LB the connections will be made as follows.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__BelbP0aQ4Q11unRMDpjTeA.png)

The client will create a connection with the LB(Load balancer) and the LB will be creating a connection with the backend server. So for the LB the client will be 10.100.5.112 and for the WSO2 server the client will be the LB. So WSO2 server will always see the clients IP as 192.168.112.8 irrespective from which client machine the request is generating from. So in-order to get the clients IP we need to read the X-Forwarded-For header, when ever a request is parsing through a proxy the clients IP will be appended to the X-Forwarded-For header. So the first IP in the X-Forwarded-For header is the IP of the request originating client.

Inorder to get the original clients IP we need to use tomcats RemoteIpValve this will read the X-Forwarded-For header and set the original clients IP as the request generating clients IP. After we get the remote clients IP we can use a Access Control valve to white list the requests. So in this case we need to use two different valves together.

Please follow instruction below,

1.  Open <WSO2\_HOME>/repository/conf/tomcat/carbon/META-INF/context.xml and add the following valve configurations,

<Valve className=”org.apache.catalina.valves.RemoteIpValve”/>  
<Valve className=”org.apache.catalina.valves.RemoteAddrValve” allow=”10\\.100\\.5\\.112"/>

2\. Then restart the server for changes to get effective.

Now if you try to access the WSO2 server with the client 10.100.5.112 you will be allows to access WSO2 servers. But for other clients the requests will be blocked. You can use what ever valve you desire in cinjuction with the RemoteIpValve.