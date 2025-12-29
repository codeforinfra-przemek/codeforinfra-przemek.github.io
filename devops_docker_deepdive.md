---
layout: default
title: "DevOps: Docker"
permalink: /ansible-desired-state.html
---

## Docker Deep Dive
### Getting started
- Docker Account to push images
- Docker Desktop on Windows (better is WSL)
- Play with docker for free labs: https://labs.play-with-docker.com/
- multipass canonical -  quick get virtyal/vm ubuntu or docker for labs
### Architecture
[https://cdn.hashnode.com/res/hashnode/image/upload/v1679764110121/5d0c2ae1-9762-404d-924d-53db207b9d2f.png?auto=compress,format&format=webp
](https://tarangsharma.hashnode.dev/docker-engine-architecture)

- client/server architecture - we send post to API on server.
- OCI - Open Container Initiative - standards for low level container images
- container - run process in isolated enviroments with resources limits (namespaces & control groups)
- - namespaces(isolation): we have multiple isolated operating system (os) like vm.
  - differences to vm: vm use the same physical machine and different os share this resources using hypervisor.
  - Containers share the same kernel on host. Container is isolated collection of namespaces (like network-net, mnt, ipc, user, pid)
- - control group control how man resources use containers on hosts,
- - layered mounts - copy and write
- Docker  - DockeAPI + containerd (liefcycle mgt) + low-level runtime (runc) for ACI
- 
