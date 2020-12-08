---
title: Apt
description: 
date: 2020-12-08T12:44:40+09:00
draft: false
weight: 2
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---


## Proxy

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

