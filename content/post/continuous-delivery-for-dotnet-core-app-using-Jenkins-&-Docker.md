+++
title = "Continuous Delivery for dotnet core apps using Jenkins and Docker"
date = "2016-08-17T11:17:34"
tags = ["Docker", "Jenkins", "dotnetcore"]
categories = ["DevOps"]
menu = ""
banner = "banners/cd-docker-jenkins.png"
+++

## Introduction

I'm very exicted with Docker & dotnet core since they're technologies changing the way we develop our applications which we haven't done as before. I'm also been attracted by the tools to build Continous Delivery process so that we can shorten the time to deliver our software to the market. I've been familiar with Release Management (from Microsoft), TeamCity, Octopus & Jenkins, which are great tools to build CI/CD solutions.

To continue investigating new way to build CI/CD solution so that we can build, ship & run using Docker images & containers, my colleague & myself have been investigating some days to have a PoC using Jenkins & Docker family toolset & it's great to have a share today.

The idea is to build a full application life-cycle management (ALM) for the project team. As a developer commits the source code to VCS (such as Gitlab/Github/VS Team Service), the changes should be built & run unit-tests immediately. If there is no issues with build & test, the code base then will be built for Docker images, then is pushed to a privated Docker Registry. Then the deployment process can continue to pull the images & deploy to a clustering environment such as Docker Swarm Mode (from Docker v1.12) or Mesos.

So in general, there should have the steps as follow:


- Assuming we have a VCS ready (I'm using Gitlab)
- Jenkins to build the source code, push images to internal Docker Registry & deploy to Docker Swarm Mode cluster 
- Internal Docker Registry (insecured mode, it's just PoC :) )
- Docker Swarm Mode to deploy services

To leverage the power of Docker, every tools listed here are Docker images, we can easily make it up by simple ```docker run``` command.

## Assuming we have a VCS ready (I'm using Gitlab)
My company has a Gitlab environment up & ready to use, I don't want to take time to build a new one. So assuming we have it available like mine.

But just in case you don't, we can easily build a new one as ```Dockerfile``` below:

```
TODO
```

## Build Jenkins Server
Here is the ```Dockerfile``` to create Jenkins Server. Be aware of the line ```RUN groupadd -g dockergroupid docker && usermod -a -G docker jenkins```, there is a ```dockergroupid``` which is a variable will be replaced by a shell script named ```create-dockerfile.sh```

Firstly take a look on the Dockerfile:

```
# Dockerfile
FROM jenkinsci/jenkins

USER root
RUN mkdir /var/log/jenkins
RUN mkdir /var/cache/jenkins
RUN chown -R jenkins:jenkins /var/log/jenkins
RUN chown -R jenkins:jenkins /var/cache/jenkins

RUN apt-get update \
      && apt-get install -y sudo \
      && rm -rf /var/lib/apt/lists/*

RUN groupadd -g dockergroupid docker && usermod -a -G docker jenkins 

USER jenkins
ENV JAVA_OPTS="-Xmx8192m"
```

And then create-dockerfile.sh

```
# create-dockerfile.sh
dockergroupid=`cat /etc/group | grep doc | cut -d":" -f3`
sed -i s,dockergroupid,$dockergroupid,g Dockerfile
```

## Build internal Docker Registry
The internal Docker Registry is neccessary to store the images built from Jenkins. Since we're building the insecured private registry, we need to update the Docker config ```/etc/default/docker```

```
# Use DOCKER_OPTS to modify the daemon startup options.
DOCKER_OPTS=" --insecure-registry 10.12.0.137:5000"
```
Please be aware that insecured registry is only used for PoC purpose & should not be used in Production environment at all.

Restart the Docker daemon service to get the update affects.

```
sudo service docker restart
```

Run the registry container

```
$ docker run -p 5000:5000 -d --name registry --restart always -d -v `pwd`/data:/var/lib/registry registry 
```

We mount a volume from Docker Host to Docker container so that we the images in the registry won't be delete as we delete / re-new the registry container.

## Buid steps on Jenkins

- Create a new free style project, put it a name ```dotnetcore test```

## 