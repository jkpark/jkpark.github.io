---
layout: post
title: 03 - General Setting
description: 우분투 18.04 LTS 기본 설정
category: ubuntu1804
---

# 1. Firewall - iptables

서버의 보안을 위해 방화벽을 설정한다. 우분투는 기본적으로 `ufw`를 라는 방화벽 관리 툴을 제공하는데, 이는 `iptables` 를 쉽게 설정하도록 하는 역활을 한다.

>The Uncomplicated Firewall (ufw) is a frontend for iptables and is particularly well-suited for host-based firewalls. ufw aims to provide an easy to use interface for people unfamiliar with firewall concepts, while at the same time simplifies complicated iptables commands to help an adminstrator who knows what he or she is doing.

`ufw`를 통해 방화벽 설정을 하는 것보다 `iptables`를 직접 다루는게 직관적이므로 `ufw`를 제거하고 `iptables`로 방화벽을 직접 설정할 것이다.

## UFW 제거

```
$ sudo apt-get purge ufw
$ sudo rm -rf /etc/ufw
$ sudo rm /etc/default/ufw
```

## iptables 초기화

`iptables` 에는 5개의 테이블이 있다.
- `raw` is used only for configuring packets so that they are exempt from connection tracking.
- `filter` is the default table, and is where all the actions typically associated with a firewall take place.
- `nat` is used for network address translation (e.g. port forwarding).
- `mangle` is used for specialized packet alterations.
- `security` is used for Mandatory Access Control networking rules (e.g. SELinux -- see this article for more details).

일반적인 경우 기본 테이블인 `filter`와 `nat`만 사용하면 된다.

```
$ sudo iptables -F
$ sudo iptables -X
$ sudo iptables -t nat -F
$ sudo iptables -t nat -X
```
`-F`는 각 체인에 설정되어 있는 규칙을 모두 제거한다. `-X`를 기본 체인을 제외한 나머지 체인을 삭제한다.

```
$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination 
```

`iptables`는 현재 시스템의 방화벽을 설정하지만 저장하지는 않는다. 즉, 서버를 재시작하면 설정한 내역이 초기화된다. 설정한 내역을 저장하기 위해서 `netfilter-persistent` 를 설치한다. `netfilter-persistent save`를 하면 `iptables` 설정이 저장된다.

```
$ sudo apt-get install netfilter-persistent iptables-persistent
```

`iptables` 설정을 저장하고 리로드한다.

```
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

재시작 후에도 설정이 잘 되어 있는지 확인한다.


## iptables 기본 설정

established sessions 허용
```
$ sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

SSH가 사용하는 TCP 22번 포트 허용
```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

localhost 허용
```
$ sudo iptables -A INPUT -i lo -j ACCEPT
```


기본 정책으로 `INPUT`과 `FORWARD`는 차단시킨다.
```
$ sudo iptables -P INPUT DROP
$ sudo iptables -P FORWARD DROP
```

이렇게 하면 앞에서 설정한 규칙의 접근만 허용되고 나머지 접근에 대해선 모두 차단된다.

설정 후에는 반드시 저장을 해야한다.

```
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

확인

```
$ sudo iptables -S
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -i lo -j ACCEPT
```


# 2. Make Own Workspace

`adduser`를 통해 사용자가 추가될 때 사용자별로 workspace를 생성하도록 한다.

생성되는 workspace는 스냅샷기능으로 관리되는 디렉토리이기 때문에 백업 및 복원이 가능하다. 백업 및 복원이 가능한 디렉토리를 `/cactus_ws1`이며 [btrfs 설치방법](02-btrfs)을 통해 다뤘다. `/cactus_ws1`에 사용자별로, 예를 들어 사용자명이 `jkpark`일 경우 `/cactus_ws1/jkpark` 을 생성하고 이 디렉토리를 `/home/jkpark/workspace`으로 링크시킬 것이다.

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

ln -s /cactus_ws1/$username $homedir/workspace
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


# 3. Change Timezone

timezone을 서울로 변경한다.

```
$ timedatectl
                      Local time: 일 2019-01-13 22:57:41 EST
                  Universal time: 월 2019-01-14 03:57:41 UTC
                        RTC time: 월 2019-01-14 03:57:41
                       Time zone: America/New_York (EST, -0500)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
$ ls -l /etc/localtime 
lrwxrwxrwx 1 root root 36  1월  3 02:18 /etc/localtime -> /usr/share/zoneinfo/America/New_York
$ timedatectl list-timezones | grep Seoul
Asia/Seoul
jkpark@cactus:~/Downloads$ sudo timedatectl set-timezone Asia/Seoul
jkpark@cactus:~/Downloads$ ls -l /etc/localtime 
lrwxrwxrwx 1 root root 32  1월 14 13:23 /etc/localtime -> ../usr/share/zoneinfo/Asia/Seoul
jkpark@cactus:~/Downloads$ timedatectl 
                      Local time: 월 2019-01-14 13:23:52 KST
                  Universal time: 월 2019-01-14 04:23:52 UTC
                        RTC time: 월 2019-01-14 04:23:52
                       Time zone: Asia/Seoul (KST, +0900)
       System clock synchronized: yes
systemd-timesyncd.service active: yes
                 RTC in local TZ: no
```



# 4. 인증서 등록

프록시 서버에 대한 인증서를 등록이 필요한 경우 인증서를 등록해준다.

먼저 인증서 파일을 가져와야 한다. USB나 여러 방법을 통해 인증서를 `/usr/local/share/ca-certificates/extra` 디렉토리에 복사한다.

```
$ sudo mkdir /usr/local/share/ca-certificates/extra
$ sudo mv samsung.crt /usr/share/ca-certificates/samsung/
$ sudo dpkg-reconfigure ca-certificates
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Processing triggers for ca-certificates (20180409) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```

