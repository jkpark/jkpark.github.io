---
layout: post
title: 11장. 파일 자동 다운로드 - flexget
description:
category: ubuntu1804
---

Flexget은 RSS를 이용해 특정 키워드가 포함된 파일을 원하는 위치에 다운받을 수 있다. 트랜스미션을 연동해서 원하는 디렉토리에 토렌트가 다운로드 되도록 할 것이다.

# 설치

```
$ sudo apt-get update
$ sudo apt-get install python3.5
$ sudo apt-get install python-pip
```

setuptool 업그레이드

```
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

# 설정 파일 작성

```
$ mkdir -p ~/.config/flexget
$ vi ~/.config/flexget/config.yml
```

<script src="https://gist.github.com/jkpark/a7ea87a3e18a506ec55d1fe1805fb5ff.js"></script>

다운로드 받고 싶은 동영상들 리스트 작성

```
$ mkdir ~/.config/flexget/wish
$ vi ~/.config/flexget/wish/list.yml
```

```
series:
  KTV:
    - '검색어'
```

# 스케줄링

`/etc/crontab` 에 아래 라인 추가.

```
# 2019-03-12 | jkpark | added below 1 line.
*/30 * * * *    jkpark  /usr/local/bin/flexget --cron execute
```

```
$ sudo systemctl restart cron
```