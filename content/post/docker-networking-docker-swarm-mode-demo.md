+++
title = "Docker networking & Docker Swarm Mode Demo"
date = "2016-07-03T13:27:20+02:00"
tags = ["Docker", "Docker Swarm", "Swarm Mode", "Docker Networking"]
categories = ["DevOps"]
menu = ""
banner = "banners/dockerswarm.png"
+++
## Introduction
This post is a step by step introduction about Docker Networking and new features of Docker Swarm Mode which has just been introduced in #DockerCon 2016. 

We will walk through by examples to understand the Docker Networking. Then will swich to Docker Swarm Mode, demo about the features such as:

- The new way to create services
- Scale the services
- Load balancing with Rounting Mesh
- Self-Healing
- Self-Organizing

## Pre-requisites
```
$ docker-machine create -d virtualbox node-0
$ docker-machine create -d virtualbox node-1
$ docker-machine create -d virtualbox node-2
```

Double check to make sure of IP Address of nodes
```
$ docker-machine ip node-0
$ docker-machine ip node-1
$ docker-machine ip node-2
```

Change **/etc/hosts** file with these above IP Addresses & nodes

## Docker Networking Demo

SSH to the node-0 node
```
docker-machine ssh node-0
```

Create a bridge network
```
docker network create -d bridge --subnet 10.10.0.0/16  vnet-1
```

Create containers within that network
```
docker run -d -p 8001:8000 --net=vnet-1 --name c1 jwilder/whoami
docker run -d -p 8002:8000 --net=vnet-1 --name c2 jwilder/whoami
docker run -d -p 8003:8000 --net=vnet-1 --name c3 jwilder/whoami
docker run -d -p 8004:8000 --net=vnet-1 --name c4 jwilder/whoami
docker run -d -p 8005:8000 --net=vnet-1 --name c5 jwilder/whoami
docker run -d -p 8006:8000 --net=vnet-1 --name c6 jwilder/whoami
```

Check the containers in the network
```
docker network inspect vnet-1
```

Ping each others
```
$ docker exec -it c1 bash
$ ping c2
$ ping c3
$ ping c4
$ # and for the others...
```

Multiple bridge networks demo
Create frontend & backend networks
```
$ docker network create -d bridge --subnet 10.11.0.0/16 frontend
$ docker network create -d bridge --subnet 10.12.0.0/16 backend
```

Remove containers off the vnet-1 network
```
$ docker network disconnect vnet-1 c1
$ docker network disconnect vnet-1 c2
$ docker network disconnect vnet-1 c3
$ docker network disconnect vnet-1 c4
$ docker network disconnect vnet-1 c5
$ docker network disconnect vnet-1 c6
```

Connect to frontend network
```
$ docker network connect frontend c1
$ docker network connect frontend c2
$ docker network connect frontend c3
```

Connect to backend network
```
$ docker network connect backend c4
$ docker network connect backend c5
$ docker network connect backend c6
```

Ping each other
```
$ docker exec -it c1 bash
$ ping c2
$ ping c3
$ ping c4
```

Connect **c3** to backend
```
$ docker network connect backend c3
$ docker exec -it c3 bash
$ ping c4
$ ping c5
```

## Docker Swarm Mode Demo

Create a swarm cluster
Initial swarm manager
```
$ docker-machine ssh node-0
$ docker swarm init
```

Node-1 joins the swarm cluster
```
$ docker-machine ssh node-1
$ docker swarm join --listen-addr 192.168.99.101:2377 192.168.99.100:2377
```

Node-2 joins the swarm cluster
```
$ docker-machine ssh node-2
$ docker swarm join --listen-addr 192.168.99.102:2377 192.168.99.100:2377
```

Go to the node-0 and check the status of nodes
```
$ docker node ls
```

Create a service
From node-0, create a service
```
$ docker service create -p 5000:5000 --name=cat mikesir87/cats
```

Testing the service by browsing at http://node-0:5000

### Scaling the service
```
$ docker service scale web=2
```

Testing the service by browsing at http://node-0:5000
Testing the service by browsing at http://node-1:5000
Testing the service by browsing at http://node-2:5000

All works, why this happens, we have only two nodes, but three endpoints are working fine? Will discuss further?...

### Routing Mesh
Ingress Load Balancing to allow the services exposing port to the outsite world. Only the services has the port published with -p option will join this **ingress network**. Otherwise, it's only added to **default_gwbridge** network.

Try to scale the services to 10 instances as follow:
```
$ docker service scale web=10
```

It's great if we have time to use a overlay user-defined network and attach the service to this network.

### Blue-print Releasing
When we need to deploy a new version, Docker 1.12 helps us to easily gain blue-print releasing with following command:
```
$ docker service update \ 
    --image trumhemcut/aspnetcore:2.0 aspnet
    --update-parallelism=2 \
    --update-delay=10
```

### Self-Healing
Go to any nodes and try to stop one container. It will create another container for us.

### Self-Organization
Try to shut down a node, it will create containers on another node for us.

### Conclusion
Hopefully the demo can illustrate the idea of how we're working with the new Docker (v1.12) & Docker Networking.