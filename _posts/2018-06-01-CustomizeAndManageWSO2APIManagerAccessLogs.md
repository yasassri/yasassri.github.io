---
title: Customize and Manage WSO2 API-Manager Access Logs
description: >-
  In any enterprise application it is important that we properly manage access
  logs which can be analyzed realtime or at a later stage. When…
date: '2018-06-01T09:38:30.045Z'
categories: [wso2, wso2apim, tomcat, devops]
keywords: [wso2]
# slug: /@ycrnet/customize-and-manage-wso2-api-manager-access-logs-e855989f9e0f
---

![](/assets/img/medium/0__cKJyAeTNvjRojkfC.jpg)

In any enterprise application it is important that we properly manage access logs which can be analyzed realtime or at a later stage. When it comes to WSO2 products they do provide access-login capabilities out of the box. But in some cases if you are using a tool to analyze the logs (e.g: splunk etc.) you will need the logs in a custom format. For example as key-value pares. So this POST explained how you can customize default logging pattern of WSO2 access logs.

In WSO2 products like WSO2 ESB, EI, APIM there are two types of access, admin operation access and API/Service invocations. If you are familiar with WSO2 products it contains 2 sets of ports, 1 for Servlets Transport (9443/9763) and one for Passthrough (8280/8243). Servlet transport is where you get admin operation requests. By default all the access logs, coming into both servlet transport and passthrough transport are written to a common access log file located in _repository/logs_ directory. But this default behavior can be changed and you can configure the logs to be written into two different files.

Access logs for requests coming into the Servlet transport is handled by the logging mechanism of the embedded tomcat server. In-order to customize the access logs of the requests which comes into the servlet transport you can refer [this](https://docs.wso2.com/display/ADMIN44x/HTTP+Access+Logging) guide.

For generating access logs for requests coming into Passthrough transport (which are mostly API/Service invocations) WSO2 uses a separate implementation where most of the code is taken from the Tomcats logging feature it-self, but the attribute set it supports differ from tomcats’. Following is how you can customize API invocation logs of the APIM nodes.

1.  First lets create access log config file. By default this file is not available in API Manager. So lets create a file named **access-log.properties** in _<PRODUCT\_HOME>/repository/conf_
2.  Add the following content to this file. (All the supported options are in the following file, uncomment them to enable as required)

\# Default access log pattern  
#access\_log\_pattern=%{X-Forwarded-For}i %h %l %u %t \\”%r\\” %s %b \\”%{Referer}i\\” \\”%{User-Agent}i\\”

\# combinded log pattern  
#access\_log\_pattern=%h %l %u %t \\”%r\\” %s %b \\”%{Referer}i\\” \\”%{User-Agent}i\\”

access\_log\_pattern=time=%t remoteHostname=%h localPort=%p localIP=%A requestMethod=%m requestURL=%U remoteIP=%a requestProtocol=%H HTTPStatusCode=%s queryString=%q  
\# common log pattern  
#access\_log\_pattern=%h %l %u %t \\”%r\\” %s %b

\# file prefix  
access\_log\_prefix=http\_gw

\# file suffix  
access\_log\_suffix=.log

\# file date format  
access\_log\_file\_date\_format=yyyy-MM-dd

#access\_log\_directory=”/logs”

3\. Now restart the server and invoke an API.

A new file will be created in the specified logging directory with the specified prefix. By default it will be created in _<SERVER\_HOME>/repository/logs_ directory. Also note that this file support rolling file appending.

Following are the attributes that are supported and these can be used in your login pattern to get request response information.

%a - Remote IP address  
%A - Local IP address  
%b - Bytes sent, excluding HTTP headers, or '-' if zero  
%B - Bytes sent, excluding HTTP headers  
%c - Cookie value  
%C - Accept header  
%e - Accept Encoding  
%E - Transfer Encoding  
%h - Remote host name (or IP address if enableLookups for the connector is false)  
%l - Remote logical username from identd (always returns '-')  
%L - Accept Language  
%k - Keep Alive  
%m - Request method (GET, POST, etc.)  
%n - Content Encoding  
%r - Request Element  
%s - HTTP status code of the response  
%S - Accept Chatset  
%t - Date and time, in Common Log Format  
%T - Content Type  
%u - Remote user that was authenticated (if any), else '-'  
%U - Requested URL path  
%v - Local server name  
%V - Vary Header  
%x - Connection Header  
%Z - Server Header

Hope this was helpful, please drop a comment if you have further queries!