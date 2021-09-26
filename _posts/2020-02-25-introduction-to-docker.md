---
toc: true
layout: post
description: A brief introduction to Docker for those with little or no background in containerization.
categories: [markdown]
title: Introduction to Docker
---

## What is Docker? 

If you've never worked with containers or VM's before, understanding what Docker is and does might be a little hard to wrap your mind around. You can start by thinking about it as a solution to a problem: you want to run code that can be tested or deployed anywhere, regardless of the machine or OS you're using. Docker fixes this problem by creating a layer on top of your machine's resources that simulates the OS you want. This way, when the layer is deployed, you know you can install whatever dependencies you need in a consistent way across machines. Therefore you only have to build an application in one way to be able to run it on any kind of machine.

## Building the Image

An **image** is an executable package that includes everything needed to run an application--the code, a runtime, libraries, environment variables, and configuration files.

A **container** is a runtime instance of an image.

A `Dockerfile` defines what goes on in the environment inside your
container. `Dockerfile`'s have the following format options:

```Dockerfile
# All Dockerfiles must specify a base image
FROM <base image>
MAINTAINER 

# Note: subsequent references to defined environment
# variables can be made by $<key> or ${<key>}
ENV <key> <value>
ENV foo /bar

# RUN executes commands on top of the current image and commits the
# result. In shell form...
RUN <shell command>
# exec form... (note that the variable isn't processed and we have
# to throw in an echo statement)
RUN ["executable", "params", "echo $SOME_VAR"]
# To use a different shell...
RUN ["/bin/bash", "-c", "echo hello"]

# Both ENTRYPOINT and CMD give you a way to identify which executable
# should be run when a container is started from your image. You must
# include at least one. CMD is easier to override and should be used
# when you want flexability in the executable that is run when
# starting the container. CMD can be used to specify default options
# for ENTRYPOINT (must use exec form for that).
CMD ["executable", "param"]

# LABELs are used to add metadata and can be viewed with the
# 'docker inspect' command
LABEL <key>=<value>
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
      
# The MAINTAINER instruction sets the Author field of the generated 
# images. The LABEL instruction is a much more flexible version of 
# this and you should use it instead
MAINTAINER <name>
LABEL maintainer="brewer36@purdue.edu"

# EXPOSE informs Docker that the container listens on the specified
# network ports at runtime
EXPOSE <port> [<port>/<protocol>...]
EXPOSE 80/tcp

# The COPY instruction copies new files or directories from <src> 
# and adds them to the filesystem of the container at the path 
# <dest>. The --chown flag requests specific ownership of the
# content added. If you want to specify a user for all RUN, CMD, and
# ENTRYPOINT instructions, use the USER command
USER <UID>[:<GID>]
COPY <src> <dest>
# or...
COPY [--chown=<user>:<group>] <src>... <dest>

# ADD is like copy but allows <src> to be a URL or automatically unpack <src> if its compressed
ADD http://example.com/foobar /

# The WORKDIR instruction sets the working directory for any RUN,
# CMD, ENTRYPOINT, COPY and ADD instructions that follow it. If a 
# relative path is provided, it will be relative to the path of the 
# previous WORKDIR instruction
WORKDIR /path/to/workdir

# defines a variable that users can pass at build-time to the builder
# with the docker build command using the 
# --build-arg <varname>=<value> flag
ARG <name>[=<default value>]
```

We use the build command to create a Docker image:

```bash
docker build -t <tagname>
```

If we want to upload this image public on a certain registry (associated with a Docker account), we tag the image as follows:

```bash
docker tag image username/repository:tag
```

and then push it to the repo:

```bash
docker push username/repository:tag
```


## Running the App

We can then run the app in the the container defined by the image:

```bash
docker run -d -p 4000:80 <tagname>
```

Note: The -d option runs the app in the background (detatched mode) and the -p maps the local port to the container's published port.

You can check to see what containers are running with...

```bash
docker container ls
```

...and they can be stopped with...

```bash
docker container stop <container name or id>
```

