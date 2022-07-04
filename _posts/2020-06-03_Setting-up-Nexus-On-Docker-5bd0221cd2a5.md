---
title: Setting up Nexus On Docker
description: >-
  In this post, I will be explaining how to set up a nexus repository with
  docker.
date: '2020-06-03T06:27:56.105Z'
categories: []
keywords: []
slug: /@ycrnet/setting-up-nexus-on-docker-5bd0221cd2a5
---

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__29xyWBC4uCA5y0Sl.jpg)

In this post, I will be explaining how to set up a nexus repository with docker.

Nexus is an artifact repository which manages software “artifacts” required for the development. If you develop software, your builds can download dependencies from Nexus and can publish artifacts to Nexus creating a way to share artifacts within an organization.

**Prerequisites:** Docker should be installed in your machine and the machine should have an active internet connection.

So let’s get started.

**Step 01**

First, we need to create a directory in the host machine to persist nexus data. If you do not do this data will be lost upon container restart.

Create a directory and set correct permissions using the following command.

mkdir /some/dir/nexus-data  
sudo chown -R 200 /some/dir/nexus-data

**Step 02**

Then start the nexus docker image with the following command.

sudo docker run -d -p 8081:8081 — name nexus -v /home/ubuntu/nexus/nexus-data:/nexus-data sonatype/nexus3:3.23.0

Then check whether the container is running

sudo docker ps

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__V5jiKKUFwVtQliw6.jpg)

Now copy the container ID from the output of the above command.

Then let’s check the logs of the container.

sudo docker logs -f <CONTAINER ID>   
or  
sudo docker logs -f nexus

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__HlktfoGQNqPfjV9i.jpg)

If you see the above logs that means the nexus server started successfully.

**Step 03**

Now in order to access the admin console you need to get the admin password. Which will be generated in the nexus-data directory the password will be stored in a file called. **/nexus-data/admin.password.**

After grabbing the password from this file navigate to the following URL.

[**http://<LocalIP or Remote IP>:8081/**](http://192.168.104.81:8081/)

And click on login and enter admin as the username and add the password you just grabbed earlier.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__43ioy__7L2uyIqndR.jpg)

Then you will be asked to complete a Wizard which will let you change the password and set a few additional configurations. And finally you should be able to see your repositories.

![](/home/yasassri/Downloads/medium-export-17fe853f8468a5f31fcccd3f4e32406ee150853a411f31fa7e2b689e994b53dc/posts/md_1656890542184/img/0__IkapqliVmgd1y7ID.jpg)

Now we have a running Nexus repository set with docker. Please drop a message if you have any questions.