---
title: Firewall
description: 
date: 2020-12-08T12:44:53+09:00
draft: false
oneliner: a network security system that monitors and controls incoming and outgoing network traffic based on predetermined security rules.
weight: 4
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---


# Overview

외부에서 홈 서버로 접근이 가능하다면 서버의 보안을 위해 방화벽을 설정하는 것이 좋다. 우분투는 기본적으로 `ufw` 를 라는 방화벽 관리 툴을 제공하는데, 이는 `iptables` 를 쉽게 설정하도록 하는 역활을 할 뿐이다.

> The Uncomplicated Firewall (ufw) is a frontend for iptables and is particularly well-suited for host-based firewalls. ufw aims to provide an easy to use interface for people unfamiliar with firewall concepts, while at the same time simplifies complicated iptables commands to help an adminstrator who knows what he or she is doing.

`ufw` 를 통해 방화벽 설정을 하는 것보다 `iptables` 를 직접 다루는게 직관적이므로 `ufw` 를 제거하고 `iptables` 로 방화벽을 직접 설정할 것이다.

# UFW 제거

```
$ sudo apt-get purge ufw
$ sudo rm -rf /etc/ufw
$ sudo rm /etc/default/ufw
```

# iptables 초기화

iptables 에는 5개의 테이블이 있다.

- `raw` is used only for configuring packets so that they are exempt from connection tracking.
- `filter` is the default table, and is where all the actions typically associated with a firewall take place.
- `nat` is used for network address translation (e.g. port forwarding).
- `mangle` is used for specialized packet alterations.
- `security` is used for Mandatory Access Control networking rules (e.g. SELinux – see this article for more details).

일반적인 경우 기본 테이블인 `filter` 만 사용하면 된다.

`-F`와 `-X`로 초기화를 시켜준다.

```
$ sudo iptables -F
$ sudo iptables -X
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

`INPUT`은 시스템으로 들어오는 패킷의 정책이고, `FORWARD`는 시스템에서 다른 시스템으로 보내는 패킷의 정책, `OUTPUT`은 시스템에서 나가는 패킷의 정책이다. 

`(policy ACCEPT)`은 아무조건도 해당되지 않을 때 ACCEPT 한다는 의미이다.

# iptables 기본 설정

## 연결된 세션(established sessions)의 INPUT 허용

```
$ sudo iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```

## loopback 허용

```
$ sudo iptables -A INPUT -i lo -j ACCEPT
```

## PING 허용

```
$ sudo iptables -A INPUT -p icmp -j ACCEPT
```

## SSH 허용

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

### SSH 특정 IP대역만 접속 허용 방법

-s IP주소 옵션으로 특정 IP주소만 접속가능 하도록 설정하여 보안수준을 높인다. `192.168.122.1` 같이 특정 IP을 입력하거나 `192.168.122.0/24` 같이 네트워크 대역을 입력할 수도 있다.

```
$ sudo iptables -A INPUT -s 192.168.122.0/24 -p tcp -m tcp --dport 22 -j ACCEPT
```


## 기본 규칙 차단

기본 규칙인 INPUT과 FORWARD는 차단시킨다.

```
$ sudo iptables -P INPUT DROP
$ sudo iptables -P FORWARD DROP
```

# 정책 확인

```
$ sudo iptables -S
-P INPUT DROP
-P FORWARD DROP
-P OUTPUT ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

# 규칙 삭제 방법

규칙 삭제는 규칙 추가 옵션인 -A를 -D로 바꿔주면 된다.

```
$ sudo iptables -D INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```


# 재부팅 시 자동으로 정책 불러오기

`iptables`는 현재 시스템의 방화벽을 설정하지만 저장하지는 않는다. 즉, 서버를 재시작하면 설정한 내역이 초기화된다. 설정한 내역을 재부팅 시 자동으로 불러오기 위해 `netfilter-persistent` 를 설치한다.


```
$ sudo apt install iptables-persistent netfilter-persistent
```


/etc/iptables 라는 디렉토리에 rules.v4와 rules.v6가 생성되었다.

netfilter-persistent save 명령어로 저장하고 netfilter-persistent reload 명령어로 리로드할 수 있다.

```
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

재부팅 후 `iptables -S` 로 정책이 유지되는지 확인한다.

# 참고

## ipv6 차단

/etc/sysctl.conf 파일을 열어 맨 밑에 아래 내용을 추가한다.

```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```