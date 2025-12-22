---
layout: default
title: "Devops CoreConcepts"
permalink: /ansible-desired-state.html
---

# Working with Terraform
main.tf: 
```
terraform {
  required_providers {
    docker = {
      source = "local/docker"
      version = "3.0.2"
    }
  }
}
 provider "docker" {    
}

data "docker_image" "flask" {
    name = "athielking/ps-flask-api"
}
```
and now work with it:
```
terraform -v
docker -v
ocker run hello-world
mkdir infrastructure && cd infrastructure
vi main.tf
terraform init
```
main.tf v2: 
```
terraform {
  required_providers {
    docker = {
      source = "local/docker"
      version = "3.0.2"
    }
  }
}
 provider "docker" {    
}

data "docker_image" "flask" {
    name = "athielking/ps-flask-api"
}

resource "docker_container" "flask_container" {
    name = "lab-api"
    image = data.docker_image.flask.id

    ports {
        internal = 8080
        external = 8080
    }
}
```
and `terraform plan` and if everything ok : `terraform apply`
to test it use: `curl localhost:8080/ping`


## better version:
```
terraform {
  required_providers {
    docker = {
      source = "local/docker"
      version = "3.0.2"
    }
  }
}
 provider "docker" {    
}

data "docker_image" "nginx" {
    name = "nginx"
}

resource "docker_container" "nginx" {
    name = "nginx"
    image = data.docker_image.nginx.id

    ports {
        internal = 80
        external = 80
    }
}

resource "docker_network" "ps_net" {
    name = "psnet"
}

volumes {
        container_path = "/etc/nginx/nginx.conf"
        host_path = "/home/pslearner/src/nginx/nginx.conf"
}
``` 
and `terraform apply --auto-approve` to avoide typing `yes`
