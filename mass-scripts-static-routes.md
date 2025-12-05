---
layout: default
title: "Masowe skrypty – zarządzanie statycznymi trasami"
permalink: /mass-scripts-static-routes.html
published: false
---

[← Back to index](/)

# Masowe skrypty – zarządzanie statycznymi trasami

## Problem

[Dlaczego musiałeś masowo zarządzać statycznymi trasami.]

## Podejście

- inwentaryzacja obecnych tras,
- generowanie zmian,
- walidacja,
- deployment.

## Przykładowy fragment skryptu

~~~python
for device in devices:
    # generowanie konfiguracji statycznych tras
    config_lines = generate_static_routes(device, routes)
    push_config(device, config_lines)
~~~

## Rezultaty

[Co udało się osiągnąć.]

## Lessons learned

[Wnioski.]
