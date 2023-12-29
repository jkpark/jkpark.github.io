---
title: SSH
description: 
date: 2020-12-08T12:44:44+09:00
draft: false
weight: 3
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---


# 설치 

원격으로 시스템에 접속하기 위해 SSH를 설치한다.

```
$ sudo apt-get install openssh-server
```

설치가 끝나면 `systemctl status sshd` 로 sshd가 동작 중임을 확인할 수 있다.

```
$ sudo service ssh status
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2020-12-17 22:49:10 EST; 1h 1min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 890 (sshd)
      Tasks: 1 (limit: 19057)
     Memory: 8.4M
     CGroup: /system.slice/ssh.service
             └─890 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups

Dec 17 22:49:10 cactus systemd[1]: Starting OpenBSD Secure Shell server...
Dec 17 22:49:10 cactus sshd[890]: Server listening on 0.0.0.0 port 22.
Dec 17 22:49:10 cactus sshd[890]: Server listening on :: port 22.
Dec 17 22:49:10 cactus systemd[1]: Started OpenBSD Secure Shell server.
```

# 설정

`/etc/ssh/sshd_config` 파일에서 포트나 AllowUsers 목록 등을 수정할 수 있다.

비슷한 이름의 `/etc/ssh/ssh_config`는 다른 시스템에 SSH연결하기 위한 설정이므로 참고하자.

설정을 마쳤으면 ssh 서비스를 재시작한다.

```
$ sudo service ssh restart
```