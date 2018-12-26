---
layout: post
title: Ubuntu18.04 - 01 - Install Ubuntu Server 18.04 LTS
description:
category: blog
tags: [Ubuntu, Ubuntu18.04]
---

# Install Ubuntu Server 18.04 LTS

## Choose Install ISO
설치 이미지는 2 종류가 있는데, Subiquity 인스톨러가 탑재된 이미지와 기존의 데비안 인스톨러가 탑재된 이미지이다.
Subiquity 인스톨러 버전은 새로운 우분투 서버 인스톨러이다.
일단 RAID 기능을 지원하지 않고 아직 불안정해보여서 기존처럼 데비안 인스톨러를 다운 받는다.
http://cdimage.ubuntu.com/releases/18.04/release/ 에서 `ubuntu-18.04.1-server-amd64.iso` 선택한다.

*Subiquity 인스톨러 버전은 https://www.ubuntu.com/download/server 에서 받으면 된다.

## Burn bootable ISO

[참고](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-ubuntu#0)


## 설치 과정

스크린샷을 찍기 위해 가상머신에 설치를 진행하였다.
디스크는 우분투가 설치될 SSD과 workspace로 사용할 HDD가 있다. 우분투 설치용 SSD는 `ext4`로 포맷하고 workspace용 HDD는 `btrfs`로 포맷할 것이다.

설치 중 중요한 내용 몇개만 설명한다. 

![select location](/images/posts/install-ubuntu1804/04.png)
패키지 저장소가 전 세계 분포되어 있다. 여기서 선택하는 지역의 서버를 기본으로 한다. 종종 한국 패키지 저장소가 다운되기 때문에 미국을 선택하였다. 미국을 선택하더라고 크게 느리지 않다.


![](/images/posts/install-ubuntu1804/05.png)
![](/images/posts/install-ubuntu1804/06.png)
![](/images/posts/install-ubuntu1804/07.png)

키보드 레이아웃은 `gnome3`를 설치할 때 수동으로 설치할 것이다.
[언어 및 키보드 레이아웃 설정](install_ubuntu1804-04-gnome3#language-and-keyboard-layout)

![](/images/posts/install-ubuntu1804/18.png)
우분투가 설치 될 SSD를 `/`에 `ext4`로 포맷시킨다.
HDD 포맷은 우분투 설치 후 [btrfs 설치방법](install_ubuntu1804-02-btrfs)를 통해 진행할 것이다.
지금은 하지 않으므로 그냥 놔둔다.

![](/images/posts/install-ubuntu1804/27.png)
프록시를 사용하기 때문에 프록시 주소를 입력한다. 설정하지 않으면 apt update가 이루어지지 않는다.

![](/images/posts/install-ubuntu1804/28.png)
패키지 업데이트는 수동으로 관리

![](/images/posts/install-ubuntu1804/29.png)
`OpenSSH server`의 경우 추 후 설치해도 되지만 귀찮으므로 지금 설치

# Static IP Configuration

[참고](https://linuxconfig.org/how-to-configure-static-ip-address-on-ubuntu-18-04-bionic-beaver-linux)

Open `/etc/netplan/01-netcfg.yaml`

```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp4s0:
      dhcp4: no
      addresses: [xx.xx.xx.xx/xx]
      gateway4: xx.xx.xx.xx
      nameservers:
        addresses: [xx.xx.xx.xx,xx.xx.xx.xx]
```

```
$ sudo netplan apply
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

# software package update and upgrade

IP과 Proxy를 설정했다면 인터넷이 가능할 것이다.
아래 명령어를 통해 설치된 소프트웨어들을 최신 버전으로 업그레이드 한다.

```
$ sudo apt-get update
$ sudo apt-get upgrade
```
