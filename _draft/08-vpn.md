---
layout: post
title: 8장. VPN
description:
category: ubuntu1804
---

# Overview

VPN(Virtual Private Network)은 외부에서 네부 네트워크 망에 접근하기 위해 쓰인다. VPN을 구축하면 외부 오는 모든 패킷이 VPN 서버를 거치게 되다. 그러므로 VPN에 접속한 컴퓨터에서 내부 네트워크에 접근할 수 있게 된다.

VPN의 방식에도 여러종류가 있는데 가장 유명한 `openVPN`으로 구축할 것이다. 그 다음 Windows 10에서 접속하기 위한 방법도 다룰 것이다.

# OpenVPN 설치

```
$ sudo apt-get update
$ sudo apt-get install openvpn
```

OpenVPN은 TLS/SSL 

https://github.com/OpenVPN/easy-rsa/releases