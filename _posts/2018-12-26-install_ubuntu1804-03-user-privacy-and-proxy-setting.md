---
layout: post
title: Ubuntu18.04 - 03 - User Privacy and Proxy Setting
description:
category: blog
tags: [Ubuntu, Ubuntu18.04, proxy,privacy]
---

# Make Own Workspace

`adduser`를 통해 사용자가 추가될 때 사용자별로 workspace를 생성하도록 한다.

생성되는 workspace는 스냅샷기능으로 관리되는 디렉토리이기 때문에 백업 및 복원이 가능하다. 백업 및 복원이 가능한 디렉토리를 `/cactus_ws1`이며 [btrfs 설치방법](install_ubuntu1804-02-btrfs)을 통해 다뤘다. `/cactus_ws1`에 사용자별로, 예를 들어 사용자명이 `jkpark`일 경우 `/cactus_ws1/jkpark` 을 생성하고 이 디렉토리를 `/home/jkpark/workspace`으로 링크시킬 것이다.

`/usr/local/sbin/adduser.local` 이라는 파일을 만들고 아래 내용을 입력한다.

```
#!/bin/sh
set -e

username="$1"
uid="$2"
gid="$3"
homedir="$4"

mkdir /cactus_ws1/$username
chown $uid:$gid /cactus_ws1/$username

ln -s /cactus_ws1/$username /home/$username/workspace
```

실행권한도 추가해준다.

```
 $ sudo chmod +x /usr/local/sbin/adduser.local
```


`adduser`로 `setu`라는 사용자를 추가해보았다. 아래와 같이 `/cactus_ws1`에 `setu`라는 디렉토리가 생성되었고, `/home/setu`에 역시 `workspace`가 링크되어 있다.

```
$ ll /cactus_ws1
total 4
drwxr-xr-x  1 root   root     20 Dec 26 21:26 ./
drwxr-xr-x 25 root   root   4096 Dec 21 01:16 ../
drwxr-xr-x  1 jkpark jkpark   24 Dec 26 03:10 jkpark/
drwxr-xr-x  1 setu   setu      0 Dec 26 21:26 setu/
$ ll /home/setu
total 20
drwxr-xr-x 2 setu setu 4096 Dec 26 21:26 ./
drwxr-xr-x 4 root root 4096 Dec 26 21:26 ../
-rw-r--r-- 1 setu setu  220 Dec 26 21:26 .bash_logout
-rw-r--r-- 1 setu setu 3771 Dec 26 21:26 .bashrc
-rw-r--r-- 1 setu setu  807 Dec 26 21:26 .profile
lrwxrwxrwx 1 root root   16 Dec 26 21:26 workspace -> /cactus_ws1/setu/
```

# General Proxy Configuration

## All User

`/etc/environment` 파일 마지막 줄에 아래 내용 추가

```
HTTP_PROXY=http://xx.xx.xx.xx:xx
HTTPS_PROXY=http://xx.xx.xx.xx:xx
http_proxy=http://xx.xx.xx.xx:xx
https_proxy=http://xx.xx.xx.xx:xx
no_proxy="localhost"
```

## apt Proxy

`/etc/apt/apt.conf` 파일에 아래 내용 추가

```
Acquire::http::Proxy "http://xx.xx.xx.xx:xx";
Acquire::https::Proxy "http://xx.xx.xx.xx:xx";
```

