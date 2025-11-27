---
layout: default
title: "DiffSync & Nautobot – modele i synchronizacja"
permalink: /diffsync-nautobot-models-sync.html
---

[← Back to index](/)

# DiffSync & Nautobot – modele i synchronizacja

## Kontekst

[Jakie dane synchronizowałeś, z jakich systemów / do jakich.]

## Model danych w Nautobot

[Jak zaprojektowałeś modele, klucze, relacje.]

## DiffSync – podejście

- definicja źródeł prawdy,
- logika porównania,
- konflikt resolution.

## Przykładowy model DiffSync

~~~python
class DeviceModel(DiffSyncModel):
    _modelname = "device"
    _identifiers = ("name",)
    _attributes = ("role", "site", "platform")
~~~

## Lessons learned

[Wyzwania i wnioski.]
