---
layout: default
title: "DevOps: Docker"
permalink: /ansible-desired-state.html
---

## DevOps Containerization and Orchestration: Deploying Containers

Base:
```
docker --version
systemctl status docker
sudo systemctl start docker
docker info
docker image ls
docker container run -p 8080:8080 --name jenkins1 jenkins/jenkins
http://localhost:8080
docker container ps -a
docker container run
docker container logs jenkins1 --follow
docker container run --rm jrottenberg/ffmpeg -version
docker container rm jenkins2
docker container stop jenkins2
docker container rm jenkins2 -f
docker container run -p 9000:8080 --name jenkins2 jenkins/jenkins
docker container run --rm -p 80:80 nginx
```

## Build and Deploy Containerized Applications

code ~/lab
```
FROM nginx
COPY custom.html /usr/share/nginx/html/index.html
```
and then: `docker image build -t custom .`
then:
```
docker image ls custom
docker container run --rm -p 80:80 custom
docker container run --rm -it mcr.microsoft.com/dotnet/sdk:8.0-alpine
dotnet --version
docker container run --rm -it -v .:/source -w /source
docker container run --rm -it -v .:/source -w /source mcr.microsoft.com/dotnet/sdk:8.0-alpine
dotnet build
docker image build -t api .
docker container run --rm -p 8080:80 api
http://localhost:8080
docker login
docker image push api
docker image pull api
```

## Use Docker Compose for Multi-container Applications

`docker container run`
compose.yaml:
```
services:
  api:
    image: api
    ports:
      - "8080:80"
    build:
      context: .
      # target: runner
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: mydb

```
and run:
```
docker container rm -f $(docker container ps -a -q)
docker container run --rm -p 8080:80 api
docker-compose up
docker container ps -a
docker-compose down
docker image rm api
docker image ls api
docker-compose build
```
### Troubleshoot changing the default Web API response and recreating its container

```
docker-compose up
docker-compose up --build
docker-compose down
docker-compose up
docker-compose exec db bash
mysql --version
docker-compose ps -a
```

# DevOps Containerization and Orchestration: Code Your Infrastructure with Kubernetes

Base:
```
minikube version
kubectl version --short
kubectl config get-contexts
minikube dashboard
kubectl get deployments
kubectl create deployment web1 --image=nginx:1.24.0-alpine-slim
kubectl get deployments
kubectl delete deployment web1
kubectl get deployments
kubectl create namespace app-cluster
kubectl get namespaces
code .
kubectl apply -f /home/pslearner/lab-resources/web1.yaml
kubectl apply -f /home/pslearner/lab-resources/web1-service.yaml
kubectl get services -n app-cluster
minikube service web1-service --url -n app-cluster
```






