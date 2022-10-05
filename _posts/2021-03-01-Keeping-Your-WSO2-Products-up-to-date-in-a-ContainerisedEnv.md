---
title: Keeping Your WSO2 Products up-to-date in a Containerised Environment
description: >-
  Nowadays, enterprises rely on a large number of applications/systems to
  full-fill their business requirements, these systems can vary from…
date: '2021-03-01T17:59:10.689Z'
categories: [WSO2, Generic]
keywords: []
tags: [wso2, docker, wso2am, wso2ei, wso2mi, wum]
image:
  path: /assets/img/medium/0__zXUtxoVAEVjiiNXY.gif
  width: 800
  height: 500
  alt:
---
Modern-day enterprises and businesses rely on a variety of integral applications/systems to fulfill their business requirements. These systems can vary from a simple HR system to a complex API Management platform. Given that businesses are highly dependent on these different underline systems that are integrated into their core business requirements, it’s crucial to keep these systems up to date.

WSO2 is one such Opensource software provider which has a variety of products to cater to different business requirements. You can read more about WSO2 from [here](https://wso2.com/). In this post, I’ll discuss how WSO2 Products can be updated within a containerized environment. In other words how WSO2 updates can be pulled to a containerized environment.

What this post will not cover is how to update a complete platform(including configurations etc.) with WSO2 Updates. This post will simply cover what’s the best strategy to pull WSO2 updated base images to your environment. Also, this post will not cover the standard Update process in a non-container environment. If you want to learn about this you can refer [this](https://updates.docs.wso2.com/en/latest/updates/overview/).

**Also, although I will be referring WSO2 MI in most examples this post is applicable to all WSO2 Products.**

#### WSO2 Releases and Updates.

In general WSO2 follows a release cycle and this can be quarterly, annually etc. depending on the product release strategy. After a product release is done, if “product bugs" or “security vulnerabilities" or “product improvements" are found, WSO2 products are patched and updated. Even after a newer version of a Product is released, WSO2 updates will be provided up until that particular product version reaches EOL. One thing to note is, WSO2 updates are available for customers with an active subscription. (For free users you can always build the product from the source if a bug is being fixed in the source :))

#### Why Update WSO2 Products?

In a nutshell “To Avoid System Failures and Vulnerabilities". Continuous maintenance of your software solution ensures system health and security throughout its lifetime. In a nutshell, the following are the benefits of frequently updating your WSO2 product.

*   Utilizing all available updates eliminates the possibility of being stymied by a known issue during your development.
*   A customer request often results in an improvement, or a fix that is built, well-tested, and delivered to you as an update.
*   WSO2 Updates give you immediate access to a surge of improvements, packaged for easy deployment into your production systems, making sure the deployment is solid and secure.
*   Update Services are available for WSO2 releases for Ten years, thus you can exploit bug and security fixes while remaining free to manage your upgrade schedule.
*   WSO2 carefully monitor hundreds of open source projects, collect and assess security reports from users or academia, run code security reviews, and automate code analysis to identify and address possible security weaknesses.

#### How would Updates Affect your Integrations.

When it comes to updates the updates can be provided to different components of the product. If you are familiar with WSO2 Micro Integrator, updates can be introduced to specific mediators, the transports layers etc. So depending on which component the updates are introduced this will directly impact your integration use cases. Hence it’s important to understand what kind of updates went in and what are the changes the users will have to do in-order to make sure the your integrations are working as expected.

#### WSO2 Update Versioning.

Inorder to come up with a proper update strategy we need to understand how WSO2 updates are versioned. When it comes to maintaining updates It’s crucial to properly version the images inorder to identify the correct update levels and the changes that are incorporated in that particular update.

Initially when the product does a GA release, the base version of the image will be versioned as 1.2.0.0.(This is WSO2 MI version) Then consecutive updates will be bumping the last digit of the versioning string. Versioning will follow a standard like the below.

**_1.2.0.0 \[GA\] — -> 1.2.0.1 \[Update\] — -> 1.2.0.2 \[Update\]_**

As shown above, the last digit of the version number increments as updates are released. This number can be used to determine the update level and the changes that went into the specific update level. Since we are using docker images to pull the updates, the docker image will carry a slightly different versioning format as shown below. The versioning will carry the base OS type as well. The following example shows the alpine based image.

**_1.2.0.0-alpine \[Latest\]< — -> 1.2.0.1-alpine \[Update\] — --> 1.2.0.2-alpine \[Update\]_**

The tags in the WSO2 docker registry will look like something below, [Official Docker Repositories — Tags (wso2.com)](https://docker.wso2.com/tags.php?repo=wso2mi)

![](/assets/img/medium/1__pasSFg8yn7jzc4ZiYomsYA.png)

As shown above, the 1.2.0.0-alpine will be the docker image tag with the latest changes. Initially, image 1.2.0.0-alpine will have the GA pack which will be continuously updated/replaced as updates are released. The Docker image metadata will consist of the actual update level of the image.

Inorer to check the metadata of the image you can use the docker inspect command. Once you have inspected the latest image the update level details will be shown as below.

![](/assets/img/medium/1__133H0394uzqjtj81E3wHNw.png)

So when we are pulling the base image from the WSO2 private repository we can read the metadata of the docker image and determine the actual update level of the latest tag. When pushing the base image to your private registry we can maintain the same update level as the tag to easily identify the update level. This is depicted in the below diagram.

![](/assets/img/medium/1__r29x8LVTTyIhilR9mJcUXw.png)

Maintaining the actual WSO2 update level internally will help the users to learn which changes have gone into the available docker image easily.

#### Checking the Update Details

Inorder to check the update details you can visit WSO2 updates Portal. [https://updates-info.wso2.com/](https://updates-info.wso2.com/)

Following is the standard update view and WSO2 is currently working on a generic view that can be used for docker updated images. I’ll update the post once this is rolled out to production.

![](/assets/img/medium/1__hMqWrYtwOVcw__k64OsdLsw.png)

Update details of a specific update will be shown as below,

![](/assets/img/medium/1__6HyJdJHPODTPOmc9UkEjpw.png)

### Implementing a Sample Pipeline

The Pipeline can simply use a DockerFile to pull the WSO2 Base image, pack necessary dependencies and push the image to the internal registry. Following is a reference Docker file that can be used to build the image.

```docker
FROM docker.wso2.com/wso2mi:1.2.0.0-alpine  
   
LABEL image="Private Base Image"  
   
# Copy resources if needed, Keystores etc.  
COPY resources/custom.jks /home/wso2carbon/wso2mi-1.2.0/  
   
# Add Remote resources if needed, common drivers etc.  
ADD http://source.file/url  /destination/path
```

Inorder to extract Docker metadata and to build the base image following script can be used. (Following was extracted from a Bamboo pipeline hence note the environment variables that are being used.)

```sh
docker pull ${bamboo.wso2DockerRegistry}:${bamboo.wso2DockerImage};  

updateLevel=$(docker inspect -f "{{json .Config.Labels.update_level }}" ${bamboo.wso2DockerRegistry}:${bamboo.wso2DockerImage} | tr -d \\")  
   
if \[ -z "$updateLevel" \]; then  
    echo "ERROR: The Update level was not found in the Image. Aboarting the process!!"  
    exit 1  
else  
    echo "The Update level is set as $updateLevel"   
fi  

cd resources/docker  
docker build -t ${bamboo.privateRegistry}/${bamboo.privateWSO2ImageName}:$updateLevel.${bamboo.buildNumber} .  

docker login ${bamboo.privateRegistry} -u ${bamboo.privateRegistryUserName} -p ${bamboo.privateRegistryPassword}  
docker push ${bamboo.privateRegistry}/${bamboo.privateWSO2ImageName}:$updateLevel.${bamboo.buildNumber}  

echo "Image ${bamboo.privateRegistry}/${bamboo.privateWSO2ImageName}:$updateLevel.${bamboo.buildNumber} pushed Successfully!"
```

The full Bamboo pipeline for the above can be found at [yasassri/wso2-baseimage-update-bamboo: Bamboo Spec repository to pull WSO2 updated Base images. (github.com)](https://github.com/yasassri/wso2-baseimage-update-bamboo)

References:

1. [https://updates.docs.wso2.com/en/latest/](https://updates.docs.wso2.com/en/latest/)