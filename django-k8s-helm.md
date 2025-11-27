---
layout: default
title: "Django / Kubernetes / Helm – od aplikacji do deploymentu"
permalink: /django-k8s-helm.html
---

[← Back to index](/)

# Django / Kubernetes / Helm – od aplikacji do deploymentu

## Kontekst

[Jaka aplikacja Django, jaki use-case.]

## Konteneryzacja

- Dockerfile dla Django,
- obraz bazowy, health checki.

## Deployment na Kubernetes z użyciem Helm

- struktura chartu,
- values dla środowisk,
- secret management.

## Przykładowy fragment `values.yaml`

~~~yaml
image:
  repository: ghcr.io/codeforinfra-przemek/my-django-app
  tag: "v1.0.0"

ingress:
  enabled: true
  hosts:
    - host: myapp.example.com
      paths:
        - /
~~~

## Observability / scaling

[Jak monitorujesz i skalujesz tę aplikację.]

## Lessons learned

[Wnioski.]
