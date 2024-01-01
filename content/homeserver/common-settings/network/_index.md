---
title: Network
description:
date: 2020-12-08T12:44:36+09:00
draft: false
weight: 1
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Static IP 설정

```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp4s0:
      dhcp4: no
      addresses: [ x.x.x.x/24 ]
      gateway4: x.x.x.x
      nameservers:
        addresses: [ x.x.x.x, x.x.x.x ]


```

# Proxy 설정

프록시를 사용한다면 프록시 설정도 한다.

\*프록시를 사용하지 않는다면 이 단계는 넘어간다.

## 환경변수에 프록시 주소 추가

프록시 주소는 `/etc/environment` 파일에 추가한다. `/etc/environment`은 모든 사용자가 로그인 시 적용되는 파일이다.

`/etc/environment` 파일을 얼어 아래 내용을 추가한다.

```
$ sudo vi /etc/environment
```

```
HTTP_PROXY=http://xx.xx.xx.xx:xx
HTTPS_PROXY=http://xx.xx.xx.xx:xx
http_proxy=http://xx.xx.xx.xx:xx
https_proxy=http://xx.xx.xx.xx:xx
no_proxy="localhost"
```

`:wq` 을 입력하고 저장 후 편집을 종료한다.

## 프록시 서버 CA 등록

프록시 서버의 인증서를 검증하기 위한 CA를 등록한다.

USB나 여러 방법을 통해 인증서를 `/usr/share/ca-certificates/extra` 디렉토리에 복사한 뒤, `sudo dpkg-reconfigure ca-certificates` 명령어로 인증서를 설치한다.

```
$ sudo mkdir /usr/share/ca-certificates/extra
$ sudo mv your.crt /usr/share/ca-certificates/extra/
$ sudo dpkg-reconfigure ca-certificates
Updating certificates in /etc/ssl/certs...
1 added, 0 removed; done.
Processing triggers for ca-certificates (20190110ubuntu1.1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
```
