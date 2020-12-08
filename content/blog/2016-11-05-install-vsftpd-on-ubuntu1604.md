---
layout: post
title: How to install vsftpd on ubuntu
description:
category: blog
tags: [Ubuntu, Ubuntu16.04, vsftpd]
---

```bash
jkpark@cactus:~$ sudo apt-get install vsftpd
다음 새 패키지를 설치할 것입니다:
  vsftpd
0개 업그레이드, 1개 새로 설치, 0개 제거 및 6개 업그레이드 안 함.
115 k바이트 아카이브를 받아야 합니다.
이 작업 후 336 k바이트의 디스크 공간을 더 사용하게 됩니다.
받기:1 http://kr.archive.ubuntu.com/ubuntu xenial/main amd64 vsftpd amd64 3.0.3-3ubuntu2 [115 kB]
내려받기 115 k바이트, 소요시간 0초 (974 k바이트/초)
패키지를 미리 설정하는 중입니다...
Selecting previously unselected package vsftpd.
(데이터베이스 읽는중 ...현재 211923개의 파일과 디렉터리가 설치되어 있습니다.)
Preparing to unpack .../vsftpd_3.0.3-3ubuntu2_amd64.deb ...
Unpacking vsftpd (3.0.3-3ubuntu2) ...
Processing triggers for systemd (229-4ubuntu11) ...
Processing triggers for ureadahead (0.100.0-19) ...
Processing triggers for man-db (2.7.5-1) ...
vsftpd (3.0.3-3ubuntu2) 설정하는 중입니다 ...
Processing triggers for systemd (229-4ubuntu11) ...
Processing triggers for ureadahead (0.100.0-19) ...
```

```bash
jkpark@cactus:~$ sudo vi /etc/vsftpd.conf
```

uncomment the below lines (line no:31, 35, 122, 123 and 125)

```
write_enable=YES
local_umask=022
chroot_local_user=YES
chroot_list_enable=YES
chroot_list_file=/etc/vsftpd.chroot_list
```

`chroot_local_user`에 대해..

`FTP`접속 시 홈 디렉토리의 상위 디렉토리 이동을 막기 위한 설정.

`Default` 상태(chroot_local_user=YES에 주석처리되어 있음)

- ftp접속 후 pwd 시 '/home/user1' 로 나옴. 상위 이동 가능

`chroot_local_user=YES` 설정

- FTP접속 후 pwd 시 '/'로 나옴. 상위 이동 불가능

`chroot_list_enable=YES` 설정

- FTP 접속 실패. 

`chroot_list` 파일 생성

- FTP 접속 시 '/'로 나옴. 상위 이동 불가능

`chroot_list` 파일 안에 `user1` 등록

- user1은 ftp 접속 시 '/home/user1'로 나옴. 상위 이동 가능

- user2는 ftp 접속 시 '/'로 나옴. 상위 이동 불가능

즉 `chroot_local_user=YES` 설정은 `ftp` 접속 시 홈 디렉토리를 root 디렉토리 처럼 인식하도록 하는 설정이다.

<http://blog.naver.com/jbells/220416391250>



Add the following lines to enable passive mode

```bash
pasv_enable=Yes
pasv_min_port=40000
pasv_max_port=40100
```

Add the following lines to enable utf8

```
utf8_filesystem=YES
```

Add the following lines to enable write permission on root directory

```
allow_writeable_chroot=YES
```

Restart vsftpd service

```bash
jkpark@cactus:~$ sudo service vsftpd restart
```

```bash
jkpark@cactus:~$ sudo iptables -I INPUT -p tcp --destination-port 40000:40100 -j ACCEPT
```

```bash
jkpark@cactus:~$ sudo iptables --list
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  anywhere             anywhere             tcp dpts:40000:40100
```

영구적으로 저장하는 방법은 <http://jkpark0r.blogspot.kr/2016/11/save-rules-of-iptables-permanently.html> 참고