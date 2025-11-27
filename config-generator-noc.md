---
layout: default
title: "Config generator dla NOC (legacy)"
permalink: /config-generator-noc.html
---

[← Back to index](/)

# Config generator dla NOC (legacy)

## Problem

[Jakie zadania NOC musiał wykonywać ręcznie.]

## Rozwiązanie

- generator konfiguracji na podstawie formularza / inputu,
- walidacja danych,
- export gotowych snippetów.

## Przykładowy fragment generatora

~~~python
def generate_interface_config(name, vlan, description):
    return f"interface {name}\n description {description}\n switchport access vlan {vlan}\n"
~~~

## Wpływ na pracę NOC

[Oszczędność czasu, mniej błędów itd.]

## Lessons learned

[Co byś zrobił inaczej dziś.]
