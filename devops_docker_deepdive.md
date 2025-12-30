---
layout: default
title: "DevOps: Docker"
permalink: /ansible-desired-state.html
---

# Docker Deep Dive

### Getting started:
- Docker Account to push images
- Docker Desktop on Windows (better is WSL)
- Play with docker for free labs: https://labs.play-with-docker.com/
- multipass canonical -  quick get virtyal/vm ubuntu or docker for labs
- 
### Architecture:

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/3d1c3693-0290-4a0a-bce3-13742cff9562" />

- client/server architecture - we send post to API on server.
- OCI - Open Container Initiative - standards for low level container images
- container - run process in isolated enviroments with resources limits (namespaces & control groups)
  - namespaces(isolation): we have multiple isolated operating system (os) like vm.
  - differences to vm: vm use the same physical machine and different os share this resources using hypervisor.
  - Containers share the same kernel on host. Container is isolated collection of namespaces (like network-net, mnt, ipc, user, pid)
  - control group control how man resources use containers on hosts,
  - layered mounts - copy and write
- Docker  - DockeAPI + containerd (liefcycle mgt) + low-level runtime (runc) for ACI
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/13417a5e-e82a-4940-9fe4-e661d068147b" />

### Images Masterclass:

- imeage is like class, and runtime is like object. 
- images are like VM templates, and runtime are like running vm.
- image is bunch of layers stuck together and defined in config yaml file.
- `docker pull redis` - to pull imahge and you see print of layers.
- we trust image/ layer base on hash it has. And compare has to esure that we got correct from internet: `docker image --digest`
- `docker info` to see more information about files system ect.
- `docker manifest inspect <redis>` to get ready yaml.
- compresed change hashes so they need to be recalculate and are named: distribution hashesh
- Good Practice:
  - use official images
  - keep images small
  - build custome images from small oficial base image
  - reference exact image tags (avoide `latest`)

### Building new images:

<img width="1188" height="1007" alt="image" src="https://github.com/user-attachments/assets/d50fdc8d-8592-492c-bfe5-bab3be082c64" />

- `Dockerfile`
- 5 layers and metadata
- `docker build -t ddd2023:nodeweb .` - `.` mean current directory
- `docker rm web -f` - to remove container
- `docker rmi ddd2023:nodeweb` - to remove iamge
  
<img width="1409" height="754" alt="image" src="https://github.com/user-attachments/assets/aa19d17e-5577-4bd6-b559-e95cf31d1a87" />
- LABEL - add metadata, not layer
- RUN - execute command and install stuff - add leyer
- COPY - create new layer, usueally to copy files
- WORKDIR - create new layers
- EXPOSE, ENTRYPOINT - metadata
- `docker build -t ddd2023:nodeweb <https://github...>` - wll load context files from git
- `docker history ddd2023:nodeweb` - to check each layer
- `docker inspect ddd2023:nodeweb` to see dockerfile

#### Multistage builds: 

<img width="1422" height="769" alt="image" src="https://github.com/user-attachments/assets/d3431332-4a20-41ed-89ac-972f92b1ceea" />
- smaller is better, reduce attack surface, only what is necessary,
- AS <name> give friendly name
- 

### Working with Containers:

<img width="1402" height="777" alt="image" src="https://github.com/user-attachments/assets/0acbf374-aa32-4b3c-bb11-e1e4fc693b47" />

- Pords wrap up container for scheduler purpose, so smallers unit is container
- container is running instance of image
- each container get his own Write/Read layer, so we can have multiple container of the same image. Container copy failes from images, and modify on r/w layer and not touch image itself <- Copy on Write. All changes are made to copy of file.
- Container virtual OS resources.
- `docker run -it <image> sh`
- `docker run -d --name web <image>` - -d detach
- `docker stop <id>`
- `docker exec <id> <command like cat filename>`
- `docker rm $(docker ps -aq) -f


### Building a Swarm:

<img width="1396" height="782" alt="image" src="https://github.com/user-attachments/assets/1e552d3a-3d0d-43f0-b505-c5bc26ac80ab" />

- clustering - take care of infrastrucuture
- swarm - handle many docker nodes. We have singe engine node, or swarm. Leader is Manager, rest are Follower. Manager take care of certificate and encryption.
- `docker swarm init` & `docker swarm joint-token manager` and save token. ssh to second docker and pass it: `docker swarm join --token...`
- odd number are best (3,5,7)
- to add workwer: `docker swarm join-token worker --rotate` and then pass on worker.
- `docker node ls`
- Orchestration - will handler big number of pods. 

### Container Network:

<img width="1130" height="457" alt="image" src="https://github.com/user-attachments/assets/121f44bf-3527-47dd-8c59-0c9f465b1cf5" />

- Bridge netowkrking (default) on NAT. Bridge docker 0. Container on one bridge can talk to each other, but not to container on second bringe. L2 issolation, so we need to map ports. 

<img width="1297" height="731" alt="image" src="https://github.com/user-attachments/assets/e6c08b4d-c355-48b3-9a73-dc4865ff4930" />
<img width="1381" height="695" alt="image" src="https://github.com/user-attachments/assets/ae1e882b-b9cf-4edc-967a-755320d88c26" />

- Multihost netowkr - single layer netowkr, all container can talk to each other, contianer to container only. We need map vlan to access world.

<img width="1265" height="740" alt="image" src="https://github.com/user-attachments/assets/a62332e6-5663-46ad-a827-ba83f9c30433" />

- MacVLAN - no bridge, no port mapping, all on wire, not for cloud. 
  
