---
title: Converting Java CLI Client to a Native Executable with GraalVM
description: Converting Java CLI Client to a Native Executable with GraalVM
date: '2022-10-16'
categories: [Programming]
keywords: [GraalVM, wso2, native-image, executable]
tags: [GraalVM, wso2]
image:
  path: /assets/img/posts/GraalVM.png
  width: 800
  height: 500
  alt:
---

GraalVM is a high-performance JDK distribution designed to accelerate the execution of applications written in Java and other JVM languages along with support for JavaScript, Ruby, Python, and a number of other popular languages. GrallVM allows you to compile your Java code into native executables which allows you to run them without a JRE. So in this post, I'll explain the steps I followed to get my CLI client to work with GraalVM. 

Some background, my CLI client was kind of a legacy application that wrapped a couple of SOAP services. Hence its dependencies required components like Axis2, Axiom, log4j etc. Initially, I used `args4j` for interactive CLI inputs and the Maven shade plugin to build an executable uber Jar.

There are a couple of ways to set up and use GraalVMs `native-image`. You can use the GraalVM Docker images, or you can simply install GraalVM on your machine. If you are super lazy you can also use Github Actions workflow with [GraalVM setup step](https://github.com/marketplace/actions/github-action-for-graalvm). But I highly recommend using the Docker image or setting it up locally which will save a lot of time when debugging issues. For this, you can refer the official [Installation Guide](https://www.graalvm.org/java/quickstart/)

The first try, I thought it was super easy. So I had my Uber Jar created and I simply executed the below, thinking everything would work.

```sh
native-image -jar capp-manager.jar
```

At least some success, with no errors when converting to a native image.

```sh
Top 10 packages in code area:                               Top 10 object types in image heap:
 663.86KB java.util                                          947.79KB byte[] for code metadata
 351.87KB java.lang                                          897.25KB java.lang.String
 273.76KB java.text                                          837.11KB byte[] for general heap data
 234.86KB java.util.regex                                    628.55KB java.lang.Class
 198.40KB com.oracle.svm.jni                                 544.92KB byte[] for java.lang.String
 193.86KB java.util.concurrent                               434.86KB java.util.HashMap$Node
 146.93KB java.math                                          224.30KB com.oracle.svm.core.hub.DynamicHubCompanion
 120.62KB java.lang.invoke                                   209.58KB java.util.HashMap$Node[]
 105.90KB com.oracle.svm.core.genscavenge                    164.34KB java.lang.String[]
  98.16KB java.util.logging                                  155.81KB java.util.concurrent.ConcurrentHashMap$Node
   1.98MB for 119 more packages                                1.54MB for 786 more object types
------------------------------------------------------------------------------------------------------------------------
                        0.3s (2.0% of total time) in 17 GCs | Peak RSS: 3.30GB | CPU load: 8.23
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/target/capp-manager-0.1.3 (executable)
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/target/capp-manager-0.1.3.build_artifacts.txt (txt)
========================================================================================================================
Finished generating 'capp-manager-0.1.3' in 14.0s.
Warning: Image 'capp-manager-0.1.3' is a fallback image that requires a JDK for execution (use --no-fallback to suppress fallback image generation and to print more detailed information why a fallback image was necessary).
```

Although there were no errors, there was a fishy warning at the end, but let's move the executable to a different location and run it. Bummer, the first error. 

```sh
Error: Could not find or load main class org.wso2.capp.client.Main
Caused by: java.lang.ClassNotFoundException: org.wso2.capp.client.Main
```

The reason is the last Warning, for some weird reason the default behavior of `native-image` command is to create a fallback image, which means if the executable fails it will execute the Jar as a fallback measure. So this Jar needs a JVM to run. This doesn't make any sense as to why they have this behavior by default, given the whole purpose of converting to a native image is to run the native image without a JVM. But I found there are legal reasons for Oracle to have this built this way. If interested you can read more from [here](https://github.com/oracle/GraalVM/issues/2648)

So let's add the `--no-fallback` flag and create the image, which will stop creating the fallback image and pack everything into the native image. 

```sh
native-image -jar capp-manager.jar --no-fallback
```

The output

```sh
------------------------------------------------------------------------------------------------------------------------
Top 10 packages in code area:                               Top 10 object types in image heap:
 706.01KB java.util                                            1.06MB byte[] for code metadata
 370.62KB java.lang                                          980.19KB java.lang.String
 274.10KB java.text                                          887.39KB byte[] for general heap data
 234.89KB java.util.regex                                    773.05KB java.lang.Class
 204.67KB java.util.concurrent                               648.69KB byte[] for java.lang.String
 202.26KB com.oracle.svm.jni                                 435.38KB java.util.HashMap$Node
 146.93KB java.math                                          282.73KB com.oracle.svm.core.hub.DynamicHubCompanion
 119.09KB java.lang.invoke                                   210.28KB java.util.HashMap$Node[]
 105.90KB com.oracle.svm.core.genscavenge                    176.99KB java.lang.String[]
  98.30KB java.util.logging                                  155.81KB java.util.concurrent.ConcurrentHashMap$Node
   2.50MB for 152 more packages                                1.60MB for 829 more object types
------------------------------------------------------------------------------------------------------------------------
                        0.4s (2.3% of total time) in 18 GCs | Peak RSS: 3.47GB | CPU load: 7.80
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/target/capp-manager-0.1.3 (executable)
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/target/capp-manager-0.1.3.build_artifacts.txt (txt)
========================================================================================================================
```

No Warning this time, so let's execute it. Once executed I got another error.

```sh
Exception in thread "main" java.lang.ExceptionInInitializerError
	at org.apache.logging.log4j.LogManager.<clinit>(LogManager.java:61)
	at org.wso2.capp.client.Main.<clinit>(Main.java:14)
Caused by: java.lang.IllegalStateException: java.lang.InstantiationException: org.apache.logging.log4j.message.DefaultFlowMessageFactory
	at org.apache.logging.log4j.spi.AbstractLogger.createDefaultFlowMessageFactory(AbstractLogger.java:246)
	at org.apache.logging.log4j.spi.AbstractLogger.<init>(AbstractLogger.java:144)
	at org.apache.logging.log4j.status.StatusLogger.<init>(StatusLogger.java:105)
	at org.apache.logging.log4j.status.StatusLogger.<clinit>(StatusLogger.java:85)
	... 2 more
Caused by: java.lang.InstantiationException: org.apache.logging.log4j.message.DefaultFlowMessageFactory
	at java.lang.Class.newInstance(DynamicHub.java:639)
	at org.apache.logging.log4j.spi.AbstractLogger.createDefaultFlowMessageFactory(AbstractLogger.java:244)
	... 5 more
Caused by: java.lang.NoSuchMethodException: org.apache.logging.log4j.message.DefaultFlowMessageFactory.<init>()
	at java.lang.Class.getConstructor0(DynamicHub.java:3585)
	at java.lang.Class.newInstance(DynamicHub.java:626)
	... 6 more
```

If we look at the above error, it's complaining about some methods not being found. Inorder to understand the reason for the above type of errors we need to understand how native images are created. When creating the native image, the `native-image` tool performs static code analysis of the code to determine the classes and methods that should be reachable in the application runtime. At the same time, GraalVM employs different strategies to improve performance. For example, given Class initialization degrades the performance in the runtime, GraalVM does something called [Class Initialization In Build time](https://www.graalvm.org/22.2/reference-manual/native-image/optimizations-and-performance/ClassInitialization/) You can read more about these on the official GrallVM website.

Looking at the above log4j error specifically, GrallVM will have trouble with file descriptors to be referenced in static fields because the files might not be present at run time. Similar issues can occur if Java Reflection is used to load classes in the runtime, native-image will not be able to predict which classes will be needed in the runtime. At this point, you can try to force `native-image` tool to pack the classes that are needed when building the image. 

Let's see how to do this. For this, you can use the `Tracing Agent` that comes with GraalVM which allows you to trace the classes being loaded in the application runtime. This agent will allow you to run your application and generate a configuration file that will tell `native-image` tool which resources/classes need to be included. The Tracing Agent is capable of detecting usages of the Java Native Interface (JNI), Java Reflection, Dynamic Proxy objects (java.lang.reflect.Proxy), or class path resources (Class.getResource). 

Let's run the client with the agent using `-agentlib:native-image-agent` flag. If you have a server-type application that keeps on running. Once run with the agent you can execute all the options in your application which will load the classes in the runtime and the agent will track them. In my case I have a CLI client which runs and exits, so I had to run the CLI client multiple times with different options that are supported and generate the configurations. One thing to note is if you run the command multiple times with the agent the configurations will be overridden, hence like in my case if you have multiple commands you may want to use multiple output directories for each command and merge the config files later. You can use something like meld to check the diff between files and merge them.  

```sh
# Option 1
java -agentlib:native-image-agent=config-output-dir=./gral-settings -jar capp-manager-0.1.3.jar list-apps --server https://localhost:9443 --username admin --password admin

# Option 2
java -agentlib:native-image-agent=config-output-dir=./gral-settings2 -jar capp-manager-0.1.3.jar download --server https://localhost:9443 --username admin --password admin --app-name HelloCompositeExporter --destination ./
.
.
.
```

After merging the files you would have multiple configuration files like below. If you see anything missing you can add them manually to each config file. 

```sh
graalvm-settings
├── jni-config.json
├── predefined-classes-config.json
├── proxy-config.json
├── reflect-config.json
├── resource-config.json
└── serialization-config.json
```

Let's build the new native image with the above config files. Since my application does http/https calls I'm passing some additional flags like `--enable-url-protocols=https,http --enable-https --enable-http` if your application doesn't need them, you can omit them. 

```
native-image -jar target/capp-manager-0.1.3.jar --no-fallback -H:ConfigurationFileDirectories=./graalvm-settings --enable-url-protocols=https,http --enable-https --enable-http
```

The output

```sh
 Top 10 packages in code area:                               Top 10 object types in image heap:
   1.49MB sun.security.ssl                                     5.82MB byte[] for code metadata
   1.04MB java.util                                            2.71MB java.lang.String
 839.46KB picocli                                              2.66MB java.lang.Class
 742.50KB com.sun.org.apache.xalan.internal.xsltc.compiler     2.57MB byte[] for general heap data
 704.42KB com.sun.crypto.provider                              2.04MB byte[] for java.lang.String
 534.36KB java.lang                                          966.80KB com.oracle.svm.core.hub.DynamicHubCompanion
 530.98KB java.lang.invoke                                   900.18KB byte[] for embedded resources
 500.51KB com.sun.org.apache.xerces.internal.impl            639.19KB java.util.HashMap$Node
 487.96KB c.s.org.apache.xerces.internal.impl.xs.traversers  561.73KB byte[] for reflection metadata
 459.58KB sun.security.x509                                  559.87KB java.lang.String[]
  19.49MB for 483 more packages                                4.69MB for 2238 more object types
------------------------------------------------------------------------------------------------------------------------
                        2.0s (4.1% of total time) in 31 GCs | Peak RSS: 6.35GB | CPU load: 8.25
------------------------------------------------------------------------------------------------------------------------
Produced artifacts:
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/capp-manager-0.1.3 (executable)
 /home/yasassri/workspace/projects/capp-manager/wso2-capp-manager/capp-manager-0.1.3.build_artifacts.txt (txt)
========================================================================================================================
Finished generating 'capp-manager-0.1.3' in 47.7s.
```

You can see the number of packages that are added to the native image is much higher than in the previous build. The more classes you add to the native image the size of the executable will increase. So you may consider optimizing it after you have successfully built the native image. After the above steps, my application started working without any further errors.

*General Tips*

- Try to use dependencies that are GraalVM compatible. In my case, I migrated from Args4J to Pocoli to build the interactive CLI wchih is compatible with GraalVM.

Hope the above helps. Please drop a comment if you have any questions. 