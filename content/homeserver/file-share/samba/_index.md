---
title: Samba
description: Ubuntu 에서 Samba 설정 방법
date: 2021-01-06T16:47:55+09:00
draft: false
weight: 0
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Overview

SMB(Server Message Block)는 윈도우즈 네트워크 파일 시스템의 기본 프로토콜 명이다. Samba는 리눅스상에서 SMB(Server Message Block) 프로토콜을 구현한 소프트웨어이기 때문에 리눅스의 파일을 윈도우PC에서 읽고 쓸 수 있게 한다.

## 공유할 디렉토리 목록

아래 표에 SMB로 공유할 디렉토리 목록을 정리했다. 각 디렉토리 별로 용도에 맞게 권한 설정도 할 것이다. (비인증 사용자(guest) 접근은 default 값으로 no로 설정 되어있다.)

|         위치         |       용도       |          권한           |
| :------------------: | :--------------: | :---------------------: |
|     /ws/\<user\>     | 개인 별 작업공간 |        All : RW         |
|       /public        |   공유 데이터    | jkpark : RW, Others : R |
| /home/jkpark/private |   개인 데이터    | jkpark : RW, Others : - |

# Samba 설치

```
$ sudo apt-get update
$ sudo apt-get install samba
```

# Samba 사용자 등록

Samba를 사용할 사용자를 등록한다. 여기서 설정하는 비밀번호는 시스템 로그인 시 사용하는 계정의 비밀번호와 연관이 없기 때문에 원하는 비밀번호로 설정한다.

```
 $ sudo smbpasswd -a jkpark
New SMB password:
Retype new SMB password:
Added user jkpark.
```

# configuration

`/etc/samba/smb.conf` 파일을 열어 공유할 디렉토리를 설정한다.

```
$ sudo vi /etc/samba/smb.conf
```

파일의 끝에 아래 내용을 추가한다.

```
[workspace]
   comment = %U's workspace
   path = /ws/%U
   browseable = no
   guest ok = no

   vfs objects = shadow_copy2
   shadow:snapdir = /mnt/ssd1_btrfs/snapshots/workspace
   shadow:format = @workspace_%Y.%m.%d_%H.%M.%S
   shadow:sort = desc

[private]
   comment = private folder for jkpark
   path = /home/jkpark/private
   browseable = no
   guest ok = no
   write list = jkpark
   valid users = jkpark
   create mask = 0600
   directory mask = 0700

   vfs objects = shadow_copy2
   shadow:snapdir = /mnt/hdd1_btrfs/snapshots/private
   shadow:format = @private_%Y.%m.%d_%H.%M.%S
   shadow:sort = desc

[public]
   comment = public folder
   path = /public
   browseable = yes
   guest ok = no
   write list = jkpark

   vfs objects = shadow_copy2
   shadow:snapdir = /mnt/hdd1_btrfs/snapshots/public
   shadow:format = @public_%Y.%m.%d_%H.%M.%S
   shadow:sort = desc

```

`shadow_copy2` 모듈이 Btrfs 스냅샷을 읽고 Windows에서 복원 가능하도록 지원한다. 포맷의 날짜형식은 건드리면 안된다.

설정 후 `systemctl restart smbd`로 삼바 재실행한다.

```
$ sudo systemctl restart smbd
```

# 방화벽 설정

`samba`가 사용하는 포트에 대한 접속은 허용해주어야 한다.

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 135 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 137 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 138 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 139 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 445 -j ACCEPT
```

확인

```
$ sudo iptables -S
...
-A INPUT -p udp -m udp --dport 138 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 139 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 445 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 135 -j ACCEPT
-A INPUT -p udp -m udp --dport 137 -j ACCEPT
...
```

저장

```
$ sudo netfilter-persistent save
```

# 윈도우PC에서 접속

윈도우PC에서 접속은 크게 두 가지가 있다. `네트워크 드라이브 연결`은 드라이브 문자(예를 들어 'C')를 할당되기 로컬 드라이브처럼 데이터 공유, 권한 등을 설정할 수 있으므로 관리에 이점이 있다. 네트워크 드라이브 연결은 사설망으로 제한되어 있다.

## restore each files

btrfs 스냅샷 자동화를 통해 주기적으로 스냅샷이 생성되기 때문에 파일의 `속성 - 이전 버전`을 누르면 복원 가능한 시점이 나올 것이다.

btrfs 스냅샷은 파일명으로 스냅샷을 찾기 때문에 파일명을 수정하거나 파일을 이동했다면 스냅샷을 불러오지 못한다. 이 경우 정확한 파일명을 알고 있다면 동일한 이름의 파일을 만들어 복원을 시도 할 수 있다.
