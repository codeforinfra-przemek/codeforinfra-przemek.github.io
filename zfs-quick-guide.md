---
layout: default
title: "ZFS – quick guide"
permalink: /zfs-quick-guide.html
---

[← Back to index](/)

# ZFS – quick guide

> This quick guide is a condensed set of notes I built while learning ZFS, heavily inspired by  
> the YouTube playlist by Learn Linux TV:  
> https://www.youtube.com/watch?v=JNYf02cRSNs&list=PLyc5xVO2uDsDI8AvG_XHptZFkFPZ4sxTQ  
> and various ZFS cheat sheets, especially:  
> - ZFS command-line reference (cheatography.com)  
> - ikrima.dev ZFS cheatsheet  

The goal here is **quick practical commands**, not a full theory book.


### Installation

On Ubuntu / Debian:

```bash
sudo apt update
sudo apt install zfsutils-linux
```

### Verify that ZFS is available:

```
zfs version
zpool version
```

### List all block devices so you know which disks you’ll use:

```
lsblk
ls -l /dev/disk/by-id

```

### Pools
A pool (zpool) is the top-level storage object in ZFS: it aggregates one or more vdevs (virtual devices) into a single storage pool. All datasets and zvols live inside a pool.

```
#Create a simple pool (single disk)
sudo zpool create tank \
  /dev/disk/by-id/ata-DISK1-ID

#List pools and basic info:
zpool list
zpool status

#Destroy a pool (⚠️ wipes all data in it):
sudo zpool destroy tank

```

### Vdev


A vdev is a building block inside a pool:
- a single disk,
- a mirror group,
- or a RAIDZ group.
The pool stripes data across vdevs, and each vdev handles its own redundancy.
Example: pool with a RAIDZ1 vdev:
```
sudo zpool create tank raidz1 \
  /dev/disk/by-id/ata-DISK1-ID \
  /dev/disk/by-id/ata-DISK2-ID \
  /dev/disk/by-id/ata-DISK3-ID

#Later you can add another vdev to grow the pool:
sudo zpool add tank raidz1 \
  /dev/disk/by-id/ata-DISK4-ID \
  /dev/disk/by-id/ata-DISK5-ID \
  /dev/disk/by-id/ata-DISK6-ID
```

### Dataset
A dataset is like a sub-filesystem inside the pool. Each dataset can have its own properties (compression, quotas, mountpoint, snapshots, encryption, etc.).
```
#Create a dataset:
sudo zfs create tank/data

#List datasets:
zfs list
zfs get mountpoint tank/data

#Change mountpoint:
sudo zfs set mountpoint=/data tank/data
```

### Data integrity
ZFS is built around data integrity:
copy-on-write: blocks are never overwritten in place; new data goes to new blocks, and metadata is updated atomically;
checksums on every block;
self-healing: on a redundant vdev (mirror/RAIDZ), if a checksum fails, ZFS reads good copies and silently repairs the bad one.
```
#You check health mainly via:
zpool status -v

#Regular scrubs walk the entire pool and verify checksums:
sudo zpool scrub tank
zpool status tank
```

### Redundancy: mirror, RAIDZ (1/2/3)

Some common layouts:
Mirror:
- zpool create tank mirror disk1 disk2
- Good read performance, simple; can lose 1 disk per mirror group.
RAIDZ1 (single parity):
- zpool create tank raidz1 disk1 disk2 disk3
- Can lose one disk.
RAIDZ2 (double parity):
- zpool create tank raidz2 disk1 disk2 disk3 disk4 disk5
- Can lose two disks.
RAIDZ3 (triple parity):
- zpool create tank raidz3 ...
- Can lose three disks.
Pick mirror vs RAIDZ based on your workload (IOPS vs capacity) and risk tolerance.

### Tuning pool & dataset parameters
```
#You can view all properties:
zpool get all tank
zfs   get all tank/data
```
Common pool-level settings:
```
# Where the whole pool is mounted
sudo zpool set altroot=/mnt tank
zpool get altroot tank
```
### Comment/description
```
sudo zpool set comment="Lab pool on old HDDs" tank
```
Common dataset-level settings:
```
# Limit dataset to 500G
sudo zfs set quota=500G tank/data
# Reserve 100G that nobody else can use
sudo zfs set reservation=100G tank/data
# Turn off atime updates for performance
sudo zfs set atime=off tank/data
```

### Compression
Compression is one of the easiest wins in ZFS.
Enable it on a dataset (recommended lz4):
```
sudo zfs set compression=lz4 tank/data
zfs get compression tank/data
```
ZFS compresses transparently; you just see less used space:
```
zfs get compressratio tank/data
```

### ZVOL (block devices)

A zvol is a block device exported from ZFS, useful for VM disks, iSCSI LUNs, or running another filesystem inside.
Create a 50G zvol:
```
sudo zfs create -V 50G tank/vm01-disk
```
You’ll get a device like:
```
ls -l /dev/zvol/tank/vm01-disk
```
You can then put e.g. ext4 on it:
```
sudo mkfs.ext4 /dev/zvol/tank/vm01-disk
```

### Managing pool & vdev health
Basic status & I/O:
```
zpool status -v
zpool iostat -v 5
```
Taking a device offline / online
```
Offline a problematic disk (for replacement):
sudo zpool offline tank /dev/disk/by-id/ata-DISK1-ID
```

Bring it back:
```
sudo zpool online tank /dev/disk/by-id/ata-DISK1-ID
```

Replacing a failed device:
```
sudo zpool replace tank \
  /dev/disk/by-id/ata-OLD-DISK-ID \
  /dev/disk/by-id/ata-NEW-DISK-ID
```

Monitor resilvering:
```
zpool status tank
```

### Importing and exporting pools
Export a pool before moving disks to another system:
```
sudo zpool export tank
```
On the new system, list pools that can be imported:
```
sudo zpool import
```
Import your pool:
```
sudo zpool import tank
```

### Deduplication

ZFS supports block-level deduplication, but you should treat it as an advanced feature:
requires a lot of RAM (rule of thumb: at least ~1–1.5 GiB RAM per TiB of logical data, sometimes more);
if you don’t have enough RAM, performance can fall off a cliff.
Enable dedup on a dataset:
```
sudo zfs set dedup=on tank/data
zfs get dedup tank/data
```

My personal rule: only enable dedup when you have a clear use case (e.g., many identical VM images) and enough RAM.
Or import all known pools:
```
sudo zpool import -a
```
