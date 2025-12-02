---
layout: default
title: "Django / Kubernetes / Helm"
permalink: /django-k8s-helm.html
---

[← Back to index](/)

# Django / Kubernetes / Helm

> **Repo:** [codeforinfra-przemek/djnago-k8s](https://github.com/codeforinfra-przemek/djnago-k8s)  
> **Stack:** Django · Docker · Kubernetes · Helm  

This project demonstrates how to take a small Django application and treat it as a **first-class cloud-native service**:

- containerized with Docker,
- deployed to a Kubernetes cluster,
- packaged and configured via Helm charts.

The goal is to have a **repeatable deployment** for a Django app that can be installed, configured and upgraded just like any other modern microservice.

---

## What this project shows

From a hiring manager’s perspective, this project is about:

- going beyond “Django on localhost” into **production-style deployment**,
- understanding how to wrap an app in a **Docker image**,
- using **Kubernetes resources** (Deployment, Service, Ingress, ConfigMap/Secret),
- and managing everything through **Helm** so the app can be installed with a single command.

---

## Architecture overview

At a high level, the architecture is:

- **Django app** (WSGI)  
  packaged into a Docker image and exposed on port 8000 inside the container.
- **Kubernetes Deployment**  
  manages replicas of the Django container (scaling up/down as needed).
- **Kubernetes Service**  
  exposes the pods inside the cluster and provides stable DNS.
- **Ingress (or LoadBalancer)**  
  exposes the app externally (cluster URL / domain).
- **Helm chart**  
  provides templated manifests and configuration via `values.yaml`.

The GitHub repo contains:

- Django project code,
- Dockerfile for building the image,
- Kubernetes / Helm manifests for deploying it to a cluster.

---

## Django application and container image

The Django part is intentionally simple: enough to show:

- URL routing,
- basic views,
- health-check style endpoint (useful for Kubernetes liveness/readiness probes).

A typical `Dockerfile` for this style of project looks like:

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /app

# Install system dependencies (if needed)
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy Django project
COPY . .

# Collect static files at build time (optional)
RUN python manage.py collectstatic --noinput

EXPOSE 8000

CMD ["gunicorn", "project.wsgi:application", "--bind", "0.0.0.0:8000"]
```
This gives you a single image that can run:
```
docker build -t djnago-k8s:local .
docker run -p 8000:8000 djnago-k8s:local
```

### Kubernetes manifests via Helm
Instead of hand-writing raw Kubernetes YAML for every environment, the project uses Helm to define a small chart.
A simplified values.yaml might look like this:
```
image:
  repository: ghcr.io/codeforinfra-przemek/djnago-k8s
  tag: "v0.1.0"
  pullPolicy: IfNotPresent

replicaCount: 2

service:
  type: ClusterIP
  port: 80
  targetPort: 8000

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: django.localdev.me
      paths:
        - path: /
          pathType: Prefix

resources:
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```
And a minimal Helm template for the Deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "djnago-k8s.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "djnago-k8s.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "djnago-k8s.name" . }}
    spec:
      containers:
        - name: django
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: 8000
          readinessProbe:
            httpGet:
              path: /health/
              port: 8000
          livenessProbe:
            httpGet:
              path: /health/
              port: 8000
```
In the real repo, the exact filenames and chart structure may differ – but the idea is the same:
values define configuration, templates define how that configuration becomes Kubernetes objects.

### Local development vs cluster deployment
The repo is designed so that:
- You can run Django locally using Docker (or directly with manage.py runserver).
- You can deploy the same app to Kubernetes with a single Helm command.

Example workflow:
1. Build and push the Docker image:
```
docker build -t ghcr.io/codeforinfra-przemek/djnago-k8s:v0.1.0 .
docker push ghcr.io/codeforinfra-przemek/djnago-k8s:v0.1.0
```
2. Deploy to Kubernetes using Helm:
```
helm upgrade --install djnago-k8s ./chart \
  --namespace web \
  --create-namespace \
  --values values.yaml
```
3.Check that pods are running:
```
kubectl get pods -n web
kubectl get svc -n web
```
4. If ingress is enabled, add an /etc/hosts entry for local testing, or use your cluster’s ingress domain.

### Configuration and secrets
The project follows standard Kubernetes best practices:
- Configuration (Django settings like debug flags, allowed hosts, etc.) can be set via environment variables in values.yaml and injected into the pod via env.
- Secrets (DB passwords, API keys) should be stored as Kubernetes Secrets and referenced from Helm values, rather than hard-coded.

Example snippet for environment variables:
```
env:
  - name: DJANGO_SETTINGS_MODULE
    value: "project.settings"
  - name: DJANGO_ALLOWED_HOSTS
    value: "django.localdev.me"
  - name: DJANGO_DEBUG
    value: "0"
```

  --restart=Never \
  -it --rm -- sh

# inside the pod
nslookup djnago-k8s.web.svc.cluster.local
curl -v http://djnago-k8s.web.svc.cluster.local
```
