---
title: How to use Windows Authentication for MSSQL with WSO2 Servers.
description: >-
  Every WSO2 product require a DB to persist different data. If we take WSO2 API
  Manager there are multiple DBs used to store different sets…
date: '2021-05-30T05:33:03.899Z'
categories: [WSO2, Generic]
keywords: []
tags: [wso2, windows, mysql]
image:
  path: /assets/img/medium/0__Mg4MOvswocDYJfUE.jpg
  width: 800
  height: 500
  alt:
---
Every WSO2 product require a DB to persist different data. If we take WSO2 API Manager there are multiple DBs used to store different sets of data, for example, APIM DB, Registry DB, User DB etc. By default these databases are pointed to a H2 Database which resides in the file system. For production deployments these databases should be externalised by pointing it to a production grade Databases like MSSQL, MYSQL, Oracle etc.

In this post I will discuss how WSO2 servers can be connected to MSSQL using Windows Authentication. For this post I will take API Manager 3.2.0 to demo the configurations.

When it comes to different authentication methods for MSSQL there are mainly following two methods.

*   SQL Server Authentication
*   Windows Authentication

When it comes to Windows Authentication there are again two methods of authentication. Kerberos based authentication and local authentication. In this post I will be explaining about local authentication. Inorder for this to work your WSO2 server needs to run on a Windows server. In this method the Server will simply use the local windows authentication to connect to the MSSQL Server.

#### So let’s get Started!!

First make sure you have enabled Windows authentication in your MSSQL Server. After this you are ready to configure the WSO2 server. Follow the steps below inorder to confiure Windows authentication.

**Step 01**

Download the MSSQL driver. You can download the relevant driver from [https://www.microsoft.com/en-us/download](https://www.microsoft.com/en-us/download)

In my case I will be using the following driver version. [https://download.microsoft.com/download/4/c/3/4c31fbc1-62cc-4a0b-932a-b38ca31cd410/sqljdbc\_9.2.1.0\_enu.zip](https://download.microsoft.com/download/4/c/3/4c31fbc1-62cc-4a0b-932a-b38ca31cd410/sqljdbc_9.2.1.0_enu.zip)

**Step 02**

Then unzip this driver archive. The content will look like something similar to below. From here on I will refer this directory as <DRIVER\_DIR>

![](/assets/img/medium/1__FkQaV286HdeodQnuiY5cYw.png)

**Step 03**

Now copy the mssql driver jar from the unziped directory to _APIM\_HOME/repository/components/lib_ directory. Make sure you select the correct driver which matches the Java runtime version you have in the VM.

**Step 04**

Next in the <DRIVER\_DIR> navigate to <DRIVER\_DIR>/_auth/<Architecture>/_ where you will see a dll file. This dll will be used by the driver to initiate the authentication. Copy the path to this .dll. (<PATH\_TO\_DLL>)

![](/assets/img/medium/1__Tu1g__NOci__oGPJuQ1ZYWag.png)

**Step 05**

Now we need to configure the path so that JDBC driver can find the above dll file. Inorder to do that open <APIM\_HOME>/bin/wso2server.bat and add the following line as shown below.

After adding the following line the batch file will look something similar to below.

![](/assets/img/medium/1__cCWY__IJsROrEBMEIl3zgSA.png)

**Step 05**

Now in the <APIM\_HOME>/repository/conf/deployment.toml change the datasource configurations as shown below. Note the additional connection parameter **integratedSecurity=true.** Also add some dummy value as the username and keep the password empty.

**Step 06**

Now start the WSO2 server by running the wso2server.bat file. If everything is in place the server should start without any errors.

That’s about it, drop a comment if you have any questions.