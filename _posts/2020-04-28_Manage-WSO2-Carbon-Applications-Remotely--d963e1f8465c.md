---
title: Manage WSO2 Carbon Applications Remotely.
description: >-
  In this post I’ll introduce you to a Java CLI client that I happened to write
  to manage WSO2 Carbon Applications. WSO2 Enterprise…
date: '2020-04-28T04:12:19.205Z'
categories: []
keywords: []
slug: /@ycrnet/manage-wso2-carbon-applications-remotely-d963e1f8465c
---

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__8O9s0QCFdnDabr__M.jpg)

In this post I’ll introduce you to a Java CLI client that I happened to write to manage WSO2 Carbon Applications. WSO2 Enterprise Integrator (was known as WSO2 ESB) has a mechanism to develop integration flows externally using Integration Studio and deploy them using .car files. This allows you to change environment specific variables and then deploy the same carbon application (Capp) between multiple environments.

### Why do we need a Client.

When we are dealing with Capps we have very limited options to manage the Capps effectively in remote servers. Although WSO2 provides a maven capp deployer plugin which is not really useful when we are dealing with proper CICD pipelines. This particular client is useful when we are writing CICD pipelines to manage integration development and deployment. This client will allow you to take backups, check the apps deployed and then properly version and deploy Capps. I will write a separate post covering how we can write a proper CICD pipeline for WSO2 EI development.

### WSO2 Carbon Application Manager.

This is a very simple client which utilizes WSO2 Admin Services to manage Carbon Applications that are deployed. The very high level architecture is something like below,

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__JdmtoHy5p16DSpm9.jpg)

#### Building the Client.

The source for this client resides at [https://github.com/yasassri/wso2-capp-manager](https://github.com/yasassri/wso2-capp-manager).

1.  First go to the repo [https://github.com/yasassri/wso2-capp-manager](https://github.com/yasassri/wso2-capp-manager) and clone it.
2.  Then go to the project root where the POM resides and execute the following command.

> mvn clean install

3\. This will create an executable uber Jar in the target directory. Refer the following screenshot.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__kY7tBfAl2foTpnQb.jpg)

#### Importing certificates to access remote servers.

In-order to access the remote servers we need to have remote server certificates in a client truststore so the SSL connections can be created with proper server name validations. Hence first we need to create a java keystore in JKS format and then import the remote servers public certificate to that keystore. You can refer to the following command to import the certificate into the keystore.

> e.g: keytool -import -alias dev-env -file public.cer -storetype JKS -keystore client-truststore.jks

After creating the keystore we can refer to this keystore when executing the client.

### Executing the client.

The Capp Manager supports the following operations at the time I’m writing this post.

*   Deploy Apps
*   Undeploy Apps
*   List Apps
*   Download Apps

Following section will explain how the above features can be used.

#### Common Parameters for the Client.

The client basically requires following parameters to execute. Following parameters are common to all the sub commands.

**— server**: used to specify the server URL

> E.g: -server [https://localhost:9443](https://localhost:9443)

**— trustore-location**: Specify the location of the client trustore.

> E.g: — trustore-location ./client-truststore.jks

**— trustore-password:** Specify the password of the trustore

> E.g: — trustore-password wso2carbon

**— username:** The server access username.

> E.g: — username admin

**— password:** Password for the server access user.

> E.g: — password admin

#### Deploy CApps

The deploy operation allows you to deploy a given carbon application.

_java -jar capp-manager-1.0.0.jar deploy — server_ [_https://localhost:9443_](https://localhost:9443) _— trustore-location ./client-truststore.jks — trustore-password wso2carbon — username admin — password admin — file ./cicd-demo-capp\_1.0.1-SNAPSHOT.car_

Output

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0____clu4KGHpxnuQjAm.jpg)
![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__acxB8lRozmF4zJNr.jpg)

If the app you are trying to deploy already exist it will throw the following error.

> _A app already exists with the name cicd-demo-capp\_1.0.1-SNAPSHOT.car_

Also you can use the — force option which will undeploy an existing carbon app and deploy the new application.

java -jar capp-manager-1.0.0.jar deploy — server [https://localhost:9443](https://localhost:9443) — trustore-location ./client-truststore.jks — trustore-password wso2carbon — username admin — password admin — file ./cicd-demo-capp\_1.0.1-SNAPSHOT.car — force

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__kJuXoF9GgkZ64vOO.jpg)

### Undeploy CApp

This operation allows you to undeploy a specified CApp. You have to specify the carbon application name only.

_java -jar capp-manager-1.0.0.jar undeploy — server_ [_https://localhost:9443_](https://localhost:9443) _— trustore-location ./client-truststore.jks — trustore-password wso2carbon — username admin — password admin — app-name cicd-demo-capp_

Output

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__wDGolhx3L7vj7SQi.jpg)

### List Apps

The list operation allows you to list all the carbon applications that are already deployed in the server.

_java -jar capp-manager-1.0.0.jar list-apps — server_ [_https://localhost:9443_](https://localhost:9443) _— trustore-location ./security/client-truststore.jks — trustore-password wso2carbon — username admin — password admin_

Output

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__5tf__jJOXR707PdaW.jpg)

If you want to get a processable output you can only read the standard out. In this case you can direct the standard error to a different stream.

_java -jar capp-manager-1.0.0.jar list-apps — server_ [_https://localhost:9443_](https://localhost:9443) _— trustore-location ./client-truststore.jks — trustore-password wso2carbon — username admin — password admin 2> /dev/null_

Output

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__x0WvF1D__jp8vq5mY.jpg)

### Download CApp

The download operation allows you to download the specified carbon application to a given location.

_java -jar capp-manager-1.0.0.jar download — server_ [_https://localhost:9443_](https://localhost:9443) _— trustore-location ./client-truststore.jks — trustore-password wso2carbon — username admin — password admin — app-name cicd-demo-capp — destination ./_

Output

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__qt1cQQm0o2HaulLC.jpg)

### Understanding the output streams

All the output logs are written into a file and to the console STD\_ERROR. All the output data is written to the console std out. We are writing outputs to the std error because the outputs that should be processable will be written to the stdout. So the output information can be easily processed. The log file will be created at <Execution\_Location>/logs/capp-client.logs

If you only wants the standard error printed in the console you can pipe the std error to a different file.

_e.g: java -jar capp-manager-0.1.2.jar help 2> /dev/null_

### Future Work

*   Read server configurations from a config file.
*   Allow the user to specify the Capp Version.
*   Allow redeploying apps through CLI.