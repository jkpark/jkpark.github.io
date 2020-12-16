---
title: Disk Setup
description: 
date: 2020-12-07T15:31:28+09:00
draft: false
weight: 0
image: "" # relative path of /static/images folder
collapse: show # show | hide | always
type: docs
---


# Overview

## sda - 128GB SSD

```
├── /boot              1GB    ext4
├── swap               8GB    swap
├── /                 40GB    ext4
└── /mnt/ssd1-btrfs   79GB    btrfs
       └── @workspace
              ├── user1
              └── user2
```

`/home/user1/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user1`.

`/home/user2/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user2`.

## sdb - 2TB HDD

```
├── /media          1024GB    ext4
└── /mnt/hdd1-btrfs 1024GB    btrfs
       ├── @public
       └── @private
              ├── user1
              └── user2
```


`/home/user1/public` will be linked to `/mnt/ssd1-btrfs/@public`.

`/home/user2/public` will be linked to `/mnt/ssd1-btrfs/@public` as same.


`/home/user1/private` will be linked to `/mnt/ssd1-btrfs/@private/user1`.

`/home/user2/private` will be linked to `/mnt/ssd1-btrfs/@private/user2`.
