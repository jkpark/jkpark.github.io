---
title: Transmission
description: Ubuntu 에서 토렌트 클라이언트 transmission 설치 방법
date: 2021-01-08T12:59:55+09:00
draft: false
weight: 1
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Overview

[transmission](https://transmissionbt.com/)은 토렌트 클라이언트이다. 오픈 소스이며 광고, 툴바같은 꾸러미를 포함하지 않기 때문에 많은 리눅스 유저들이 사용하고 있다. 이 포스트에서 설치할 `Transmission-daemon`은 클라이언트지만 Web Interface를 지원하여 웹 브라우저를 통해 접속이 가능하다.

# 설치

```
$ sudo apt update
$ sudo apt install transmission-daemon
```

# 환경설정

## download 폴더 및 watch 설정

transmission은 특정 디렉토리에 토렌트 파일을 넣으면 자동으로 토렌트가 추가되게 할 수 있다.

다운로드할 디렉토리와 watch 디렉토리를 만든다.

```
$ sudo mkdir /public/torrent/downloads /public/torrent/watch
```

보안을 높이기 위해 소유자와 그룹을 바꿔준다.

```
$ sudo chown -R debian-transmission: /public/torrent
```

사용자 `jkpark`이 해당 디렉토리의 파일에 권한을 얻기 위해 그룹에 추가한다.

```
$ sudo usermod -a -G debian-transmission jkpark
$ sudo chmod -R g+w /public/torrent
```

## 토렌트 다운로드 완료 시 토렌트 리스트 제거 스크립트

토렌트 다운로드 완료 시 토렌트가 리스트에서 제거하는 스크립트를 작성한다.

```
$ sudo vi /usr/local/etc/transmission_post_download.sh
```

<script src="https://gist.github.com/jkpark/da82772ad1d50e98945be1d70790b9c0.js"></script>

```
$ sudo chown ebian-transmission: /usr/local/etc/transmission_post_download.sh
$ sudo chmod 700 /usr/local/etc/transmission_post_download.sh
```

환경설정 시 script-torrent-done-filename에 작성한 스크립트의 경로를 넣어주면 된다.

## 환경설정 파일 작성

transmission은 중지된 상태에서 설정파일을 수정해야 되므로 데몬을 중지한다.

```
$ sudo systemctl stop transmission-daemon.service
```

`/etc/transmission-daemon/settings.json` 파일을 열어 아래와 같이 설정한다.

<script src="https://gist.github.com/jkpark/f97b7809b564f77d39134b41228f6a45.js"></script>

`rpc-password`, `rpc-username`는 접속에 사용할 ID와 비밀번호이다.

`script-torrent-done-filename`은 토렌트 다운로드가 끝나면 실행할 스트립트이다. 위에서 생성한 `/usr/local/etc/transmission_post_download.sh`가 실행되도록 한다.

'"umask": 2`는 다운로드되는 디렉토리와 파일의 권한이다. 2로 설정하면 각각 775, 664의 권한을 갖는다.

` "watch-dir": "/public/torrent/watch",`는 watch 디렉토리 설정이다.

서비스 실행

```
$ sudo systemctl start transmission-daemon.service
```

# 방화벽 설정

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 51413 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 51413 -j ACCEPT
```

# 리버스 프록시

웹 브라우저로 `http://localhost:9091` 에 접속하면 transmission의 웹 인터페이스에 접속할 수 있다. 홈 서버의 경우 CLI모드이기 때문에 localhost로의 접속이 불가능하므로, 어디에서든 웹 인터페이스에 접속하기 위해 리버스 프록시를 설정한다.

```
$ sudo vi /etc/nginx/sites-available/exsample.com
```

서버블록에 아래 내용 추가

```
location /transmission {
    client_max_body_size 0;
    proxy_pass http://127.0.0.1:9091;
    proxy_http_version 1.1;
    proxy_pass_header X-Transmission-Session-Id;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

웹 서버 재시작

```
$ sudo nginx -t
$ sudo systemctl restart nginx
```

이제 웹 브라우저에서 `https://exsample.com/transmission` 으로 접속할 수 있다.

# WEB-UI 변경

https://github.com/ronggang/transmission-web-control
