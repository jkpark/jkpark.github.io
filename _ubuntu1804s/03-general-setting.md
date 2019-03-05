---
layout: post
title: 3장. 기본 설정
category: ubuntu1804
---


# 인터넷 연결 설정

우분투 서버가 설치되었다면 가장 먼저 IP 설정을 해주어야 한다. 대부분의 경우 IP주소는 공유기의 DHCP 기능을 통해 동적으로 할당되고 언제든지 바뀔 가능성이 있기 때문에 IP주소가 바뀌지 않도록 설정하도록 한다.

시스템에 인터넷 선이 잘 연결되어있는지 확인 후 아래 명령어를 입력하여 내 네트워크의 상태를 확인한다.

```
$ ip addr
```

현재 시스템의 네트워크 인터페이스가 출력될 것이다.

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:93:eb:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.74/24 brd 192.168.122.255 scope global dynamic ens3
       valid_lft 2112sec preferred_lft 2112sec
    inet6 fe80::5054:ff:fe93:eb53/64 scope link 
       valid_lft forever preferred_lft forever
```

`ens3`이 외부 인터넷 망과 연결되는 인터페이스다. 내 가상머신의 IP주소는 `192.168.122.74`로 할당되었다. 공유기를 사용 중이라면 일반적으로 IP 주소가 `192.168`로 시작한다. IP주소 뒤에 붙은 `/24`는 네트워크 클래스를 의미한다. 가정에서 공유기는 일반적으로 C클래스인 24를 사용한다.

## IP 설정

우분투 18.04 에서는 `netplan`을 통해 네트워크를 설정한다. 고정 IP 주소 설정을 위해 `/etc/netplan` 디렉토리에 있는 `01-netcfg.yaml` 파일을 수정해야 한다. (Subiquity 인스톨러로 우분투 서버를 설치하였다면 `50-cloud-init.yaml` 파일을 수정한다.)

```
$ sudo vi /etc/netplan/01-netcfg.yaml
```

아래와 같이 DHCP를 통해 네트워크를 설정하도록 되어있다.

```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: yes
```

`ens3` 네트워크 인터페이스를 고정 IP으로 설정하기 위해 자신의 환경에 맞게 내용을 입력한다.

가산머신에는 아래와 같이 설정할 것이다.

- IP주소 : 192.168.122.3
- Gateway : 192.168.122.1
- DNS 서버 : 192.168.122.1

결과는 아래와 같아야 한다. 공백(들여쓰기) 또한 정확하게 입력해야 한다.
```
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    ens3:
      dhcp4: no
      addresses: [192.168.122.3/24]
      gateway4: 192.168.122.1
      nameservers:
        addresses: [192.168.122.1]
```

파일 수정을 마쳤다면 `:wq`를 입력하여 저장 후 편집을 종료한다. `:`은 명령어모드이고 `w`는 저장, `q`는 종료를 의미한다.

아래 명령어를 통해 수정사항을 적용한다.
```
$ sudo netplan apply
```

설정에 문제가 없다면 `ip addr` 명령어를 입력하였을 때 IP주소가 `192.168.122.3`으로 바뀐 것을 볼 수 있다.

```
$ ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:93:eb:53 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.3/24 brd 192.168.122.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe93:eb53/64 scope link 
       valid_lft forever preferred_lft forever
```

## Proxy 설정

프록시를 사용한다면 프록시 설정도 한다. (프록시를 사용하지 않는다면 이 단계는 넘어간다.)

### 환경변수에 프록시 주소 추가

프록시 주소는 `/etc/environment` 파일에 추가한다. `/etc/environment`은 모든 사용자가 로그인 시 적용되는 파일이다.

`/etc/environment` 파일을 연다.
```
$ sudo vi /etc/environment
```

아래 내용을 추가한다.
```
HTTP_PROXY=http://xx.xx.xx.xx:xx
HTTPS_PROXY=http://xx.xx.xx.xx:xx
http_proxy=http://xx.xx.xx.xx:xx
https_proxy=http://xx.xx.xx.xx:xx
no_proxy="localhost"
```

`:wq` 을 입력하고 저장 후 편집을 종료한다.

### apt Proxy

우분투는 `apt` 명령어를 통해 소프트웨어 관리를 다룰 수 있다.

`/etc/apt/apt.conf` 파일을 연다. (파일이 없다면 생성될 것이다.)
```
$ sudo vi /etc/apt/apt.conf
```

아래 내용을 추가한다.

```
Acquire::http::Proxy "http://xx.xx.xx.xx:xx";
Acquire::https::Proxy "http://xx.xx.xx.xx:xx";
```

`:wq` 을 입력하고 저장 후 편집을 종료한다.



### 인증서 등록

프록시 서버에 대한 인증서를 등록이 필요한 경우 인증서를 등록해준다.

먼저 인증서 파일을 가져와야 한다. USB나 여러 방법을 통해 인증서를 `/usr/local/share/ca-certificates/extra` 디렉토리에 복사한다.

```
$ sudo mkdir /usr/local/share/ca-certificates/extra
$ sudo mv samsung.crt /usr/local/share/ca-certificates/extra/
$ sudo dpkg-reconfigure ca-certificates
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Processing triggers for ca-certificates (20180409) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```


# 패키지 업데이트

IP와 프록시를 설정했다면 인터넷 연결이 되었을 것이다. 우분투 서버에 설치되어 있는 패키지를 최신으로 업그레이드하기 위해 먼저 `apt-get update`로 최신 패키지정보를 받아온다.

```
$ sudo apt-get update
```

우분투 서버에 설치되어 있는 패키지들을 최신 버전으로 업그레이드를 한다.

```
$ sudo apt-get upgrade
```

# 미국 미러 아카이브를 카카오 미러 아카이브로 변경

`/etc/apt/sources.list` 파일과 `/etc/apt/sources.list.d` 디렉토리 안에 있는 파일에 아카이브 주소들이 적혀있다. 패키지 관리는 
먼저 `/etc/apt/sources.list` 파일 내용을 출력하여 Source 리스트를 확인한다.
```
$ cat /etc/apt/sources.list
# 

# deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

#deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://us.archive.ubuntu.com/ubuntu/ bionic main restricted
# deb-src http://us.archive.ubuntu.com/ubuntu/ bionic main restricted

...

```

우분투 서버 설치 과정 중 지역을 미국으로 선택했다면 아카이브 Source가 `http://us.archive.ubuntu.com/ubuntu/` 일 것이다. `apt-get upgrade`를 했을 때 느꼈겠지만 미국에 있는 아카이브를 통해 소프트웨어를 다운받았기 때문에 속도가 다소 느렸다. 카카오 에서 제공하는 우분투 미러 아카이브로 바꿔준다.

`/etc/apt/sources.list` 파일을 백업한다.
```
$ sudo cp /etc/apt/sources.list /etc/apt/sources.list.ori
```

`sed` 명령어를 통해 `us.archive.ubuntu.com` 를 `mirror.kakao.com`로 바꿔준다.

```
$ sudo sed -i -e 's/us.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
```

`/etc/apt/sources.list` 파일을 출력하면 아래와 같이 `mirror.kakao.com` 로 바낀 것을 확인할 수 있다.
```
$ cat /etc/apt/sources.list
# 

# deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

#deb cdrom:[Ubuntu-Server 18.04.2 LTS _Bionic Beaver_ - Release amd64 (20190210)]/ bionic main restricted

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirror.kakao.com/ubuntu/ bionic main restricted
# deb-src http://mirror.kakao.com/ubuntu/ bionic main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirror.kakao.com/ubuntu/ bionic-updates main restricted
# deb-src http://mirror.kakao.com/ubuntu/ bionic-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirror.kakao.com/ubuntu/ bionic universe
# deb-src http://mirror.kakao.com/ubuntu/ bionic universe
deb http://mirror.kakao.com/ubuntu/ bionic-updates universe
# deb-src http://mirror.kakao.com/ubuntu/ bionic-updates universe

...

```


`apt-get update` 시 http://mirror.kakao.com/ubuntu을 통해 업데이트 되는 것을 볼 수 있다.
```
$ sudo apt-get update]($ sudo apt-get update
Get:1 http://mirror.kakao.com/ubuntu bionic InRelease [242 kB]
Get:2 http://mirror.kakao.com/ubuntu bionic-updates InRelease [88.7 kB] 
Get:3 http://mirror.kakao.com/ubuntu bionic-backports InRelease [74.6 kB])

...

```

# timezone 설정

우분투 설치 시 인터넷 연결이 되어있지 않았다면 타임존을 수동으로 설정하였을 것이다. 자동으로 타임존이 잡혔다면 이 단계는 넘어가도 된다.

`timedatectl` 명령어를 통해 현재 타임존을 출력한다.

```
$ timedatectl 
                      Local time: Tue 2019-02-22 03:25:40 EST
                  Universal time: Tue 2019-02-22 08:25:40 UTC
                        RTC time: Tue 2019-02-22 08:25:41
                       Time zone: US/Eastern (EST, -0500)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```

타임존을 서울로 변경하기 위해 타임존 리스트를 본다.

```
$ timedatectl list-timezones | grep Seoul
Asia/Seoul
```

타임존을 서울로 변경한다.

```
$ sudo timedatectl set-timezone Asia/Seoul
```

# OpenSSH 설치

원격으로 시스템에 접속하기 위해 SSH를 설치한다. 

```
$ sudo apt-get install openssh-server
```

설치가 끝나면 `systemctl status` 로 `sshd`가 동작 중임을 확인할 수 있다.

```
$ sudo systemctl status ssh
[sudo] password for jkpark: 
● ssh.service - OpenBSD Secure Shell server
   Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-02-24 23:58:33 EST; 1min 58s ago
  Process: 1055 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
 Main PID: 1067 (sshd)
    Tasks: 1 (limit: 4603)
   CGroup: /system.slice/ssh.service
           └─1067 /usr/sbin/sshd -D

Feb 24 23:58:33 cactus systemd[1]: Starting OpenBSD Secure Shell server...
Feb 24 23:58:33 cactus sshd[1067]: Server listening on 0.0.0.0 port 22.
Feb 24 23:58:33 cactus sshd[1067]: Server listening on :: port 22.
Feb 24 23:58:33 cactus systemd[1]: Started OpenBSD Secure Shell server.
```

외부에서 홈 서버로 접속이 가능하다면 누군가 무차별 공격을 시도할 수 있다. 그러므로 기본포트를 사용하지 않고 다른 임의의 숫자의 포트로 설정하기도 한다. 나는 포트를 `8822`로 변경하고 접속가능한 사용자도 설정하였다.

`/etc/ssh/sshd_config` 파일을 연다.

```
$ sudo vi /etc/ssh/sshd_config
```

`Port` 옵션의 주석을 해제하고 `8822`로 설정한다.
바로 아래 `AllowUsers` 항목을 추가하고 접속허용할 사용자명을 적는다.

```
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 8822
AllowUsers jkpark
```

`:wq` 을 입력하고 저장 후 편집을 종료한다.

설정을 마쳤으면 ssh 서비스를 재시작한다.

```
$ sudo systemctl restart ssh.service
```
