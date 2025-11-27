---
layout: default
title: "ZFS – quick guide"
permalink: /zfs-quick-guide.html
---

[← Back to index](/)

# ZFS – quick guide

> Ten krótki przewodnik powstał m.in. na bazie materiałów z kursu:  
> [ZFS Tutorial (All Parts)](https://www.youtube.com/watch?v=JNYf02cRSNs&list=PLyc5xVO2uDsDI8AvG_XHptZFkFPZ4sxTQ)

Celem tego wpisu jest **szybki start z ZFS**: stworzenie puli, datasetów, snapshotów oraz sprawdzanie stanu systemu plików. Przykłady zakładają system Linux (np. Ubuntu) i dyski przeznaczone tylko dla ZFS.

---

## 1. Instalacja ZFS

Na Ubuntu / Debian:

~~~bash
sudo apt update
sudo apt install zfsutils-linux
~~~

Sprawdzenie wersji:

~~~bash
zfs version
~~~

---

## 2. Identyfikacja dysków dla ZFS

Przed stworzeniem puli dobrze jest upewnić się, które urządzenia chcemy użyć.

Wyświetlenie dysków:

~~~bash
lsblk
~~~

lub bardziej szczegółowo:

~~~bash
sudo fdisk -l
~~~

W praktyce warto używać **stałych identyfikatorów**:

~~~bash
ls -l /dev/disk/by-id
~~~

Przykład ID dysku:

```text
/dev/disk/by-id/ata-Samsung_SSD_860_EVO_500GB_S3Z0NB0K123456

