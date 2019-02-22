---
layout: post
title: 13장. Docker
description: Docker란 OS Level의 가상화를 하는 툴이다. 다른 말로 컨테이너화(Containerzation)이라고도 한다. 어플리케이션과 어플리케이션을 구동하는데 필요한 툴, 라이브러리, 설정 등을 하나의 컨테이너로 구축할 수 있고 각 컨테이너는 독립적으로 실행되어 개별 시스템을 사용할 때와 동일한 환경으로 운용되어 보안도 강화된다.
category: ubuntu1804
english: false
---

# Overview

Docker란 OS Level의 가상화를 하는 툴이다. 다른 말로 컨테이너화(Containerzation)이라고도 한다. 어플리케이션과 어플리케이션을 구동하는데 필요한 툴, 라이브러리, 설정 등을 하나의 컨테이너로 구축할 수 있고 각 컨테이너는 독립적으로 실행되어 개별 시스템을 사용할 때와 동일한 환경으로 운용되어 보안도 강화된다.
예를 들어 nginx, php-fpm, db를 각 컨테이너화 시키면 3개의 개별 시스템을 사용하는 것와 동일한 환경이 된다. 그러므로 해커가 시스템에 침투하여도 container 안에서의 액션 밖에 할 수가 없다.

이 포스트에서는 Ubuntu18.04 에서 Docker를 설치하여 container를 다루는 방법을 살펴볼 것이다.


# Installation

Ubuntu18.04에서는 Docker 저장소를 등록하여 `apt-get install`로 docker를 설치할 수 있다.

## Set up the repository

Docker GPG 키 등록

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

stable 저장소 등록

```
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

## Install docker-ce

```
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

Docker CE 가 설치되고 실행되었을 것이다.

```
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
  Drop-In: /etc/systemd/system/docker.service.d
           └─http-proxy.conf, https-proxy.conf
   Active: active (running) since Wed 2019-01-23 16:54:04 KST; 1 weeks 0 days ago
     Docs: https://docs.docker.com
 Main PID: 1951 (dockerd)
    Tasks: 24
   CGroup: /system.slice/docker.service
           └─1951 /usr/bin/dockerd -H fd://
```


# Manage Docker as a non-root user

Docker를 설치하면 docker 그룹이 생성되지만 아무 사용자도 그룹에 속해있지 않다. 즉, 일반 사용자가 docker 명령어를 시키기 위해선 `sudo`로 root권한을 빌려야 한다. 일반 사용자가 `sudo` 없이 docker 명령어를 사용하기 위해 docker 그룹에 일반 사용자를 추가한다.

나는 사용자 `jkpark`을 docker 그룹에 추가시켰다.

```
sudo usermod -aG docker jkpark
```

로그아웃 후 다시 로그인하여 로그인한 유저의 그룹정보를 갱신한다.

`sudo` 없이 docker 명령어를 실행시켜 본다.

```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete 
Digest: sha256:2557e3c07ed1e38f26e389462d03ed943586f744621577a99efb77324b0fe535
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

...

```

`docker run hello-world`는 `hello-world`라는 이미지를 실행시키는 명령어이다. docker를 설치하고 처음으로 이 명령어를 실행시키면 `hello-world` 이미지가 local에 없기 때문에 default storage engine에서 `hello-world` 이미지를 찾아 다운로드되고 실행된다.

> 참고 https://docs.docker.com/install/linux/linux-postinstall/

# Configuration proxy

> 참고 https://docs.docker.com/config/daemon/systemd/



# How to

## 비인증 저장소 추가 방법

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

##  정지상태 컨테이너들 전부 삭제

```
$  docker rm `docker ps -f status=exited -q`
```


## 불안전한 이미지 삭제

독커 이미지를 만드는 도중 실패하면 `<none>` 형태의 이미지가 만들어지는 경우가 종종 발생한다.
이런 이미지를 삭제한는 명령어는 아래와 같다.

```
$ docker rmi `docker images -f "dangling=true" -q`
```
