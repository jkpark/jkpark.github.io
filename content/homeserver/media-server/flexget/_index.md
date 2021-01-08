---
title: Flexget
description: Flexget 설치 및 설정 방법
date: 2021-01-08T13:00:02+09:00
draft: false
weight: 2
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

Flexget은 여러가지 일을 자동화하는 툴이다.

RSS feed로 부터 특정 키워드가 포함된 토렌트를 검색하여 원하는 위치에 토렌트 다운로드 받을 것이다. flexget에서 transmission연동을 지원한다.

# 설치

설치 준비

```
$ sudo apt-get update
$ sudo apt-get install python3.5
$ sudo apt-get install python-pip
$ sudo pip install --upgrade setuptools
```

flexget 설치

```
$ sudo pip install flexget
```

트랜스미션 플러그인 설치

```
$ sudo pip install transmissionrpc
```

확인

```
$ flexget --version
2.21.18
Latest release: 3.1.1
```

# 설정

내 설정은 아래와 같다.

- secrets.yml : 계정 정보같은 개인정보 저
- config.yml : Flexget 설정
- wish.yml : 다운로드할 목록

각 파일은 `.config/flexget`에 생성하면 된다.

```
$ mkdir -p ~/.config/flexget
$ touch secrets.yml
$ touch config.yml
$ touch wish.yml
```

참고로 내 설정은 https://github.com/jkpark/flexget_config 에 올려두었다.

## secrets.yml

<script src="https://gist.github.com/jkpark/e0a6df7e6a67146c483cd65a1ed238aa.js"></script>

folder를 알맞게 입력한다.

transmission과 연동하기 위해 username과 password 를 알맞게 입력한다.

## config.yml

<script src="https://gist.github.com/jkpark/53a1808bf0376b9db71c93a9c3828748.js"></script>

가져올 rss정보를 넣는다.

## wish.yml

<script src="https://gist.github.com/jkpark/b1e11e08746cd1a084d7a74abd279d16.js"></script>


# 스케줄링

crontab에 추가하여 자동으로 flexget이 실행되도록 한다.

*crontab 참고 https://crontab.guru/

```
$sudo vi /etc/crontab
```

```
# 2019-03-12 | jkpark | added below 2 line.
10 0-2,6,18-23 * * *    jkpark  /usr/local/bin/flexget --cron execute --tasks download_*
12 6,18,0 * * * jkpark  /usr/local/bin/flexget --cron execute --tasks download2_movie
```
