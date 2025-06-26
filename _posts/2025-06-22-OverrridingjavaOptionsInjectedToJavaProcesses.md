---
title: Solving Datadog Java Agent Conflicts in Kubernetes With A Simple C Library
description: Solving Datadog Java Agent Conflicts in Kubernetes With A Simple C Library
date: '2025-06-22'
categories: [Kubernetes, Datadog, Java]
keywords: [Kubernetes, WSO2, Clustering, Datadog, Java, JavaTools]
tags: [Kubernetes, WSO2, Datadog, Java]
image:
  path: /assets/img/posts/kb_dd.png
  width: 800
  height: 400
  alt: ''
---
# Solving Datadog Java Agent Conflicts in Kubernetes With A Simple C Library

Recently, I ran into an issue where Datadog's admission controller was automatically injecting `JAVA_TOOL_OPTIONS` into our Java app containers, causing startup issues. This Java app container was executing some other Java commands before the actual app, and the injected `JAVA_TOOL_OPTIONS` was causing issues for these other Java runs. In this post, I'll walk you through how I solved this with a lightweight shared library approach.

## The Problem

When you deploy applications in K8s clusters with Datadog monitoring enabled, there are different ways to integrate Datadog. In a Java app, you have to install the Java agent in the container and pass the agent configs to the JVM, typically through the `JAVA_TOOL_OPTIONS` environment variable. If you have enabled the Datadog Admission Controller in your cluster, Datadog will use a Mutating Webhook and mutate your pod to inject Datadog dependencies and configs. The Datadog init containers used to just inject the `JAVA_TOOL_OPTIONS`, which was available to you in the container runtime. In other words, if you `exec` into a pod and `echo $JAVA_TOOL_OPTIONS`, you should see the variable. So if you needed, you had the capability to unset this if you didn't want it passed to certain operations happening on container startup. But with the latest Datadog versions, they took it to another level, where the Datadog init container now injects a Linux shared library and sets their library in `/etc/ld.so.preload` to inject this variable, which made it impossible to remove from environment variables using traditional methods. Shared libraries can be either passed through the `LD_PRELOAD` variable or by setting the library in `/etc/ld.so.preload`.

### What are shared libraries, AKA LD_PRELOAD?

Before diving in further, let me explain what `LD_PRELOAD` does. It's a Linux feature that allows you to load shared libraries before any other libraries when a program starts. This means:

1. **Library Loading Order**: Libraries specified in `LD_PRELOAD` are loaded first.
2. **Function Overriding**: You can override functions from other libraries.  
3. **Early Execution**: Constructor functions in preloaded libraries run before the main application starts.

The difference between `LD_PRELOAD` and `/etc/ld.so.preload`:
- **LD_PRELOAD**: Environment variable, affects only the current process.
- **/etc/ld.so.preload**: System-wide file, affects all processes.

## Solution: Fight Fire with Fire!

Instead of fighting with Kubernetes configurations or changing application code, I decided to create my own shared library that automatically unsets the problematic `JAVA_TOOL_OPTIONS` environment variable. The key was understanding how `LD_PRELOAD` works and using it to my advantage. Since Datadog was using `/etc/ld.so.preload`, their library was loading system-wide. But by using `LD_PRELOAD`, I could ensure my library loads after Datadog's but before the Java application starts. 

### The Solution

The implementation is surprisingly simple—just 12 lines of C code that uses the GNU C library's constructor attribute:

```c
#define _GNU_SOURCE
#include <stdlib.h>
#include <stdio.h>

__attribute__((constructor))
static void unset_java_tool_options(void) {
    if (getenv("JAVA_TOOL_OPTIONS")) {
        unsetenv("JAVA_TOOL_OPTIONS");
        printf("JAVA_TOOL_OPTIONS has been unset.\n");
    }
}
```

The magic happens with the `__attribute__((constructor))` directive—this ensures the function runs automatically when the shared library is loaded. Here's the execution order:

1. **System starts the Java process**
2. **Datadog's library loads** (from `/etc/ld.so.preload`) and sets `JAVA_TOOL_OPTIONS`.
3. **My library loads** (from `LD_PRELOAD`) and unsets `JAVA_TOOL_OPTIONS`.  
4. **Java application starts** with the variable unset.

This way, I'm essentially running my unset script after Datadog runs its script, but before the Java process is initialized.

## Building the Library

If you just want to use the compiled library, you can locate the source code and a compiled artifact here: https://github.com/yasassri/shared-lib/actions/runs/15891248186

When building the library, you need to make sure the library is compatible with the environment. In my case, I needed to compile this for our production environment which runs on x86_64 Linux. Here's how you can build it:

### Local Compilation (Linux x86_64)
```bash
gcc -shared -fPIC -o unset_java_tool_options.so unset_java_tool_options.c
```

### Cross-platform Compilation with Docker

Since I was working on a Mac with Apple Silicon, I used Docker to ensure I got the right architecture. The key option here is `--platform linux/amd64`. This approach works great for anyone not on Linux:

**For Mac users:**
```bash
docker run --platform linux/amd64 --rm -v "$(pwd)":/src ubuntu:20.04 sh -c \
  "apt-get update && apt-get install -y gcc && cd /src && \
   gcc -shared -fPIC -o unset_java_tool_options.so unset_java_tool_options.c"
```

**For Linux users:**
```bash
docker run --rm -v "$(pwd)":/src ubuntu:20.04 sh -c \
  "apt-get update && apt-get install -y gcc && cd /src && \
   gcc -shared -fPIC -o unset_java_tool_options.so unset_java_tool_options.c"
```

I chose this Docker approach because it:
- Uses Ubuntu 20.04 to match our container base images.
- Targets x86_64 architecture for production compatibility.  
- Gives me a consistent build environment regardless of my local machine.

## Using It In Production

The key is using `LD_PRELOAD` to load our library for the specific process:

```bash
LD_PRELOAD=./unset_java_tool_options.so java -jar your-application.jar
```

This approach works because:
- Datadog's system-wide library sets the variable.
- Our process-specific library unsets it.  
- The Java application gets a clean environment.

### Container Integration

In our Dockerfile, I integrated it like this:

```dockerfile
# Copy the shared library
COPY shared-libs/unset_java_tool_options.so /opt/app/
```

You can simply COPY this to your Docker base image and use it whenever you need it. Also note: if you set this globally, Datadog will never run, even for the intended processes. 

## Why This Approach Works Well

I really like this solution because it's:

1. **Non-invasive**: No changes needed to our application code.
2. **Automatic**: Works transparently at runtime.  
3. **Lightweight**: Minimal overhead with a tiny shared library.
4. **Flexible**: Can easily enable or disable by changing the `LD_PRELOAD` variable.

## Architecture Notes

I compiled this specifically for x86_64 architecture because:
- Our production K8s clusters run on x86_64 nodes.
- WSO2 containers typically use x86_64 base images.  
- Cross-compilation ensures compatibility regardless of where I'm developing.

## When I'd Use This Solution

This approach is perfect when:
- You need to disable Datadog's automatic Java agent injection for specific processes within your container.
- Legacy applications have compatibility issues with the injected agent.  

## Wrapping Up

This simple shared library shows how a well-targeted solution can solve complex environment conflicts in containerized applications. By using the `LD_PRELOAD` mechanism and constructor functions, I was able to cleanly intercept and modify the runtime environment without touching any application code.

If you're facing similar environment variable conflicts with Java applications, give this approach a try. Drop a comment if you have any thoughts or questions. 

---