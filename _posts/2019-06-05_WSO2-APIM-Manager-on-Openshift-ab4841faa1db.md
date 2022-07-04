---
title: WSO2 APIM Manager on Openshift
description: >-
  In this post I will be explaining how WSO2 products can be deployed on top of
  Openshift. By the time I was writing this blog I was using…
date: '2019-06-05T04:27:05.694Z'
categories: []
keywords: []
slug: /@ycrnet/wso2-apim-manager-on-openshift-ab4841faa1db
---

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__vhZV2kNvYHsY6yVYZ9MRGw.jpeg)

In this post I will be explaining how WSO2 products can be deployed on top of Openshift. By the time I was writing this blog I was using **Openshift 3.11** and this deployment is hardcoded for **MSSQL.**

Openshift is the commercial Kubernetes offering that is provided by RedHat. Although the underline runtime is Kubernetes there are few differences when it comes to Openshift from Kubernetes. Mainly traffic routing, scaling/rollbacks and deployments are different in Openshift. Feature wise the main deferences are,

1\. Routes  
2\. Deployment Configs  
3\. Image streams etc.

Routes are similar to ingress controllers in Kubernetes and routes are responsible for exposing the services externally. Deployment configs are similar to Deployments in Kubernetes but the internally it operated differently. It is true that we can use most of the kubernetes  
configurations within openshift as it is, but using Kubernetes artifacts as they are won’t let you utilize the added features that are available in Openshift.  
In order to utilize the features that are provided by openshift you should use openshift specific object types instead of Kuberenetes objects.  
For example you can use Kubernetes deplyments instead of Openshifts Deployment configs but using Deployment configs will allow  
you to use some neat features that are provided by Openshift, like adding triggers to deployments, undoing deployments, monitor deployment process and add triggers to your deployment to perform rolling updates etc. Some of these capabilities are not available with Kubernetes Deployments.

In this deployment I will be focussing on doing a production ready deployment, where you will do proper backing up of logs etc.

The related artifacts are located in [https://github.com/yasassri/openshift-wso2-apim](https://github.com/yasassri/openshift-wso2-apim) From here on I will refer this repository as <Artifact\_Home>

#### Understanding the repository structure.

There are several directories in the <Artifact\_Home>. Mainly docker, k8s, scripts, and resources. You can see the repository structure in the below image.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__9UTyqNAlrlnsbvOOj9Fyhg.png)

In the following section what each directory contains will be explained.

#### Docker artifacts

Docker directory contains the docker artifacts to create docker images that are required by the openshift environment. If we look closely at the docker folder, it has the following content.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__x2RPhCYw8__vt32tFbgkgxQ.png)

Each product/profile has its own docker resources to create the necessary docker images. So if we consider a single directory which represents a single product it has the following folder structure.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__k__ch7lB7v2Ygj34qJNfbzg.png)

Files directory contains the necessary external artifacts that the Docker build process requires, for example, any external dependencies (SQL drivers) that the wso2 servers require should be added to the lib directory. The init.sh script is the entry point of the created docker container. So if you want to perform a task on docker container startup you should add this to the init.sh script. More details on docker image creation will be in a different section of this document.

#### K8S Artifacts

Then we have the k8s directory. Which contains all the artifacts that are required to setup WSO2 servers on top of openshift. If we look closely it has the following folder structure.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1____8tMSEaTnfhd0uSF88JimQ.png)

This structure is different from the docker structure since the grouping is done based on deployment patterns and a deployment contains multiple products and should be started in a specific order to make sure the deployment starts without errors and it is successful, so the products/profiles are grouped in a logical manner. If we take the apim-is directory it contains, WSO2 API Manager, Identity Server, API Manager Analytics worker and IS Analytics worker related artifacts. The folder structure would look like following,

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__uyo7s__rg0MhvMObYeH____Ow.png)

Each directory named as the product will have K8S artifacts for that product. Typically it will have a K8S deployment yaml, a service yaml and in some cases a volume claim. As an example refer the following,

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__FCpOkdBPw32TO1Lr.jpg)

Then for the configurations maps, we use the confs directory which will contain the configuration files that are needed for the config maps. For each product it will have a directory in the configs folder.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__hM4l74mw1oGxWaD8.jpg)

The above configurations will be mounted to the container on startup to alter the default configurations of WSO2 products. Only the configuration files that needs alterations are included here. These will be mounted as secrets or config maps depending on the requirement. The configurations doesn’t have all the values hardcoded, most configuration files will have place holders in them and these will be populated when deploying. For example if we take the _carbon.xml_ the hostname of the server will be populated when deploying. So the configuration file will have a placeholder like below.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__tIyXpTZTkni6LR1__.jpg)

More information on these placeholders and how they get replaced will be provided in the later section of the document.

Then in the folder structure we have a routes directory which will contain the openshift route configurations. Following are the routes of the apim-is deployment.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__1xE21lWdWQ9zeu64.jpg)

Then within the K8S folder we have a directory called rbac and volumes.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__d7cPnDw4PsejJWZRm__wEug.png)

Rbac directory has the access control configurations that are required by the cluster user to get clustering information. The volumes has the configurations to create persistent volumes.

#### Resources for the Deployment

The **_resources_** directory holds the Db scripts and other relevant resources that are needed for the deployment.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__MFT__XbsG6VfbJ97Xj0zcWw.png)

Then we have the scripts directory which contains all the automation scripts to prepare the deployment artifacts and then to deploy WSO2 servers. How to use these scripts will be explained later.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__EL0jTsdsDrb8__YTAuySF5Q.png)

### Deploying WSO2 servers

### Creating necessary Databases and Tables for WSO2 servers

WSO2 servers require DBs to persist the application and user data, so we need credentials to access a database. This particular DB user should have and Read/Right access to table and Table create permissions with the created Databases.

In the <REPO\_HOME>/resources/db-setup directory it has the following scripts.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__uifQfZ__xcnuI6trXPt__t6Q.png)

1.  First inorder to create the necessary databases execute the _1-databases.sql_ script on the MSSQL Server. This will create the necessary DBs.
2.  Then execute all the scripts within the subfolders to create the necessary tables in the DBs. For _example apim/mssql.sql, common\_um/mssql.sql_. There is no order for the execution, we need to execute all the scripts.
3.  Executing the above scripts will create the necessary databases along with the tables. Now create a user who has aforementioned permissions to access the above databases.

#### Creating the key-stores for the servers

You can follow the steps in the below section to create a new keystore with a private key and a new public certificate. We will be using the keytool that is available with your JDK installation for creating the keys-stores. Note that the public key certificate we generate for the keystore is self-signed. Therefore, if you need a public key that is CA-signed, you need to generate a CA-signed certificate and import it to the keystore.

Change the CN name appropriately depending on the environment and execute the following command in the terminal. You can specify any value for the <KEY\_PASS>

keytool -genkey -alias wso2carbon -keyalg RSA -keysize 2048 -keystore wso2carbon.jks -dname “CN=\*.apps.wso2.com, OU=BP,O=WSO2,L=EC,S=WS,C=EC” -storepass <KEY\_PASS> -keypass <KEY\_PASS> -validity 7300

Here we are using \*.apps.wso2.com as the CN for the self signed certificate. You need to change this depending on your environment. <KEY\_PASS> is the only value you should be changing. Also note that the storepass and the keypass has to be the same.

Next fom the created keystore above lets extract the public certificate from the keystore. To do that execute the following command. This will store the public key in a file called wso2pub.pem.

keytool -export -alias wso2carbon -keystore wso2carbon.jks -file wso2pub.pem

Now let’s import this to the client-trustore.jks of WSO2 server. From any WSO2 pack copy the client-trustore.jks and from where the keystore is located execute the following command to delete the existing public certificate. When prompted for the password enter wso2carbon as the password.

keytool -delete -alias wso2carbon -keystore client-truststore.jks

Then execute the following command to import the new public certificate we created.

keytool -import -alias wso2carbon -file wso2pub.pem -keystore client-truststore.jks -storepass <KEY\_PASS>

Next execute the following command to change the default password of the client-trstore. When prompted for the existing password enter wso2carbon as the password.

keytool -keystore client-truststore.jks -storepass wso2carbon -storepasswd

You can refer the following documentation for more information on using an existing private key and a certificate to create the necessary keystores. [https://docs.wso2.com/display/ADMIN44x/Creating+New+Keystores#CreatingNewKeystores-Creatinganewkeystore](https://docs.wso2.com/display/ADMIN44x/Creating+New+Keystores#CreatingNewKeystores-Creatinganewkeystore)

### Downloading WSO2 Product distributions and updating

For the deployment we need WSO2 distributions, there are few ways to download WSO2 distributions. We will use WSO2 Update Manager to download distributions in this case.

Inorder to download wso2 product distributions or any updates you need to setup WSO2 Update manager(WUM) tool. You can refer the following document to setup the WUM tool ([https://wso2.com/wum/download](https://wso2.com/wum/download)). So you should have an active subscription to WSO2 products and a user to configure WUM. Also an internet connection is required by the WUM tool.

After setting up WUM you can download the products by executing the following command.

> wum add wso2is-analytics-5.7.0

We need to download the following set of products and you can repeat the above command with the following values.

*   wso2is-km-5.7.0
*   wso2am-analytics-2.6.0
*   wso2am-2.6.0
*   wso2is-analytics-5.7.0

After downloading you can list the products with the following command,

> wum list

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__WsTlh0wbxLp237h__.jpg)

Next you can check whether wum updates are available for a given product. You can execute the following command to check for updates.

> wum check-update wso2ei-6.4.0

This will give the following output if updates are available.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__5__gerzSygIvSirjN.jpg)

Now you can do an update of the product. To do this execute the following command.

> wum update wso2ei-6.4.0

You can repeat the above steps and get updates for all the products.

After updating the product the product should be available at a location similar to following.

> <USER\_HOME>/.wum3/products/wso2am/2.6.0/full

If you perform multiple updates you will have multiple distributions in this location so make sure you use the distribution with the latest timestamp. Refer the following image.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__E0t0r1vBrkTyqS7S.jpg)

**Preparing Openshift Environment**

Before we start deploying WSO2 artifact we need to prepare Openshift environment. In the openshift environment we need to create a project, service account etc. This needs to performed by the Openshift administrator. The Openshift administrator can refer the script [1-prepare-openshift.sh](https://github.com/yasassri/openshift-wso2-apim/blob/master/scripts/1-prepare-openshift.sh) The content of this file is as following,

#!/bin/bash  
  
\# ------------------------------------------------------------------------  
\# Copyright 2019 WSO2, Inc. (http://wso2.com)  
#  
\# Licensed under the Apache License, Version 2.0 (the "License");  
\# you may not use this file except in compliance with the License.  
\# You may obtain a copy of the License at  
#  
\# http://www.apache.org/licenses/LICENSE-2.0  
#  
\# Unless required by applicable law or agreed to in writing, software  
\# distributed under the License is distributed on an "AS IS" BASIS,  
\# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  
\# See the License for the specific language governing permissions and  
\# limitations under the License  
\# ------------------------------------------------------------------------  
oc=\`which oc\`  
  
\# This is for creating the secret to authenticate with the Docker registry.  
DOCKER\_REG\_URL="docker.wso2.com"  
PROJECT\_NAME=$K8S\_NAMESPACE  
DOCKER\_REG\_USERNAME="DOCKER\_REG\_PASSWORD"  
DOCKER\_REG\_EMAIL="dev@wso2.com"  
  
oc new-project $PROJECT\_NAME --description="WSO2 Dev Project" --display-name="WSO2 Dev"  
oc project $PROJECT\_NAME  
  
oc create serviceaccount wso2svc-account -n $PROJECT\_NAME  
  
\# Create the pvs   
\# Most cases this will be created by the cluster admin, so DO NOT execute if already there.  
oc create -f ../k8s/volumes/persistent-volumes.yaml  
  
\# Create the Rbac for WSO2 clustering  
oc create -f ../k8s/rbac/rbac.yaml  
  
\# Creating the  ADM policy to retrieve k8s cluster information from the svc account  
  
\# We need to create a Docker registry secret if authentication is required by the registry.  
#oc create secret docker-registry wso2creds --docker-server=${DOCKER\_REG\_URL} --docker-username=${DOCKER\_REG\_USERNAME} --docker-password=${DOCKER\_REG\_PASSWORD} --docker-email=${DOCKER\_REG\_EMAIL}

#### Creating the Docker images and Pushing

Note: You need to have docker installed in the system. Also unzip as well.

After preparing the openshift environment and copying all the WSO2 distributions to a single directory we can start the docker image creation process. Note that I’m using the internal docker registry of Openshift here and you can modify these scripts and use any docker registry that is desired.

Next navigate to the <Artifact\_Home>/scripts directory and open the 0-setEnvironment.sh and set the variables with appropriate values. The content of this file is as below and each variable will be explained below,

Following variable name contains the project name/namespace of the WSO2 deployment, this name is specific to openshift and can be set depending on the environment.

export K8S\_NAMESPACE=”wso2”

This is the docker registry URL of the Openshift environment, you can ask your openshift admin for this.

export DOCKER\_REGISTRY\_URL=”registry.apps.wso2.com”  
export DOCKER\_REGISTRY\_NAMESPACE=$K8S\_NAMESPACE

The name of the directory which contains the keystores that are being used

export KEYSTORE\_DIR\_NAME=”dev-env” # The directory name which contains environment specific keystores in resources/keystores  
export WSO2\_PACK\_LOCATION=”” # The directory where WSO2 packs reside

Following are the DNS names that are used for different WSO2 servers. Need to change the values depending on the environment.

#DNS Names for the servers  
export APIM\_HOST\_NAME=”apim.apps.wso2.com”  
export APIM\_GW\_HOST\_NAME=”gw.apps.wso2.com”  
export IS\_HOST\_NAME=”identity.apps.wso2.com”

Following are the database details of WSO2

#DB Details  
#For all the DB’s the same user will be used.  
export DB\_URL=”10.2.1.2:1433" # host:port  
export DB\_USER=”root”  
export DB\_USER\_PASSWORD=”root123456"

WSO2 admin credentials.

#Master admin of the WSO2 server  
export ADMIN\_USER=”admin”  
export ADMIN\_USER\_PASSWORD=”bpadmin”  
export ADMIN\_USER\_PASSWORD\_ENCODED=”YnBhZG1pbg==” # This is the base64 encoded value of the admin password, you can use [https://www.base64encode.org/](https://www.base64encode.org/) to encode.

WSO2 keytstore credentials

#Keystore Details  
export KEYSTORE\_PASSWORD=”wso2carbon”  
export TRUSTORE\_PASSWORD=”wso2carbon”

After properly setting the variables source this file. Execute the following command to do this.

> source ./0-setEnvironment.sh

After sourcing the file execute the docker image creation script.

> ./2-createDockerImages.sh

This will create the docker images. To push the docker images to the registry execute the following script.

> ./3-pushDockerImages.sh

After pushing the docker images you can browse the docker images in the Openshift console. You need to login to the openshift console and you need to have proper permissions to view the created project in Openshft. After login select the project and navigate to Builds -> Images tab and the created images will be shown as below.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__fV3MzWTtvJyNiuk7__zFAZg.png)

#### Deploying APIM

Before deploying the products we need to bootstrap configurations, in order to do this execute the following command from within the scripts directory.

> ./4-bootstrap-configs.sh

Next we need to execute the apim deployment script. You can execute the following script from with the scripts directory.

> ./5-apim-is-deploy.sh

The above will create the Openshift artifacts that are related to API Manager and IS. You can check the deployment process by navigating to the Openshift console. The progress of the deployment will be shown in the Openshift console.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__LJVfk57o__m5wLMhtw2cdlQ.png)

To check the status of the pods you can click on a deployment or you can open the pods page under Applications.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__xFZJclgnabOLkVzS9OWnSw.png)

Also from the pod you can check the events that are associated with the pod. This will reveal any errors that are generated in the deployment time.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__Nri9C1NXjR7d7Fda4OPoSA.png)

Also from the pods you can check the logs of the pods by navigating to the logs tab,

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__xLDg0FPmRKxN9ggOvn8ALQ.png)

If you need to access the pod itself you can navigate to the terminal tab and it will open a terminal session to the pod

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__eHt____XG999mMIA__pNBdE__Q.png)

#### Accessing the Deployment

Openshift routes handle all inbound traffic that is received by the application. In-order to check the status of routes you can navigate to the routes section. The accessible URLs of the deployment will be shown in routes. You can navigate to a specified route to access the different components of APIM.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/1__qd6Y87EkoXYxq9UdvR06__Q.png)

So that’s it hope this helps and please drop a comment if you have any questions.