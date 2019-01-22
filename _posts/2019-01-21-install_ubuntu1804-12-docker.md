---
layout: post
title: Ubuntu18.04 - 12 - Docker
description: 
category: blog
tags: [Ubuntu, Ubuntu18.04, docker]
english: false
---

> https://docs.docker.com/install/linux/docker-ce/ubuntu/ 참고하여 작성함.

# PPA 등록

키 등록

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

저장소 등록

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

인스톨

```
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

# 현재 사용자를 docker 그룹에 추가

> Docker CE is installed and running. The docker group is created but no users are added to it. You need to use sudo to run Docker commands.
> 
현재 로그인한 사용자가 `docker` 명령어를 `sudo`없이 실행가능하도록 하기 위해 `docker`그룹에 현재 로그인한 사용자를 추가한다.

```
sudo usermod -aG docker jkpark
```

로그아웃 후 다시 로그인한다.

> 참고 https://docs.docker.com/install/linux/linux-postinstall/

# Proxy

> 참고 : https://docs.docker.com/config/daemon/systemd/

# 비인증 저장소 추가 방법

```
# vi /etc/docker/daemon.json

{
   "insecure-registries" : ["xx.xx.xx.xx"]
}
```

재시작

```
# systemctl daemon-reload
# systemctl restart docker
```