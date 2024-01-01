---
title: Time / Date
description: Ubuntu 20.04 Focal Fossa에서 timezone, ntp 설정하는 방법
date: 2020-12-08T12:44:48+09:00
draft: false
weight: 6
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Timezone

우분투 설치 시 인터넷 연결이 되어있지 않았다면 타임존을 수동으로 설정하였을 것이다. 자동으로 타임존이 잡혔다면 이 단계는 넘어가도 된다.

`timedatectl` 명령어를 통해 현재 타임존을 출력한다.

```
$ timedatectl
               Local time: Sat 2020-12-19 00:24:26 EST
           Universal time: Sat 2020-12-19 05:24:26 UTC
                 RTC time: Sat 2020-12-19 05:24:26
                Time zone: America/New_York (EST, -0500)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

타임존을 서울로 변경하기 위해 타임존 리스트를 본다.

```
$ timedatectl list-timezones | grep Seoul
Asia/Seoul
```

타임존을 서울로 변경한다.

```
$ sudo timedatectl set-timezone Asia/Seoul
```

확인

```
$ timedatectl
               Local time: Sat 2020-12-19 14:26:00 KST
           Universal time: Sat 2020-12-19 05:26:00 UTC
                 RTC time: Sat 2020-12-19 05:26:00
                Time zone: Asia/Seoul (KST, +0900)
System clock synchronized: no
              NTP service: active
          RTC in local TZ: no
```

# NTP

정확한 시간을 동기화하기 위새 NTP를 이용한다.

## 설치

```
$ sudo apt-get install ntp
```

공식 목록 http://www.pool.ntp.org/ko/zone/kr 에서 서버목록을 확인한다.

```
server 3.kr.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org
```

`/etc/ntp.conf` 파일을 열어 서버를 추가한다.

```
# Specify one or more NTP servers.
server 3.kr.pool.ntp.org
server 1.asia.pool.ntp.org
server 2.asia.pool.ntp.org
```

## 방화벽 설정

```
$ sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT
$ sudo netfilter-persistent save
```

## 확인

서비스를 재시작한다.

```
$ sudo systemctl restart ntp
```

ntp 동기화를 확인한다.

```
$ sudo ntpq -p
     remote           refid      st t when poll reach   delay   offset  jitter
==============================================================================
 0.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 1.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 2.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 3.ubuntu.pool.n .POOL.          16 p    -   64    0    0.000    0.000   0.000
 ntp.ubuntu.com  .POOL.          16 p    -   64    0    0.000    0.000   0.000
-211.233.40.78   125.185.190.74   2 u   29   64   37    3.772    6.502   2.554
+ott129.hkcable. 118.143.17.82    2 u   38   64   37   47.540    5.049   2.128
-ntp.hkg10.hk.le 130.133.1.10     2 u   35   64   37   44.548   39.918   4.120
*106.247.248.106 141.223.182.106  2 u   26   64   37    5.429    4.140   2.985
+211.233.84.186  216.239.35.12    2 u   21   64   37    3.290    4.562   3.311
-ec2-13-209-84-5 17.253.114.253   2 u   25   64   17   29.177   15.007   6.481
+49.247.128.87 ( 106.247.248.106  3 u   25   64   37    4.708    4.212   3.323
```

NTPQ Command row output interpretation:

- `-` Discarded by the cluster algorithm.
- `+` Included in the combine algorithm. This is a good candidate if the current server we are synchronizing with is discarded for any reason.
- `*` The current system peer. The computer is using this remote server as its time source to synchronize the clock
  Ref: https://pthree.org/2013/11/05/real-life-ntp/

또한, 우분투의 디폴트 NTP service가 `NTP service: n/a` 으로 disable 된 걸 확인한다.

```
$ timedatectl
               Local time: Thu 2020-12-17 14:45:22 KST
           Universal time: Thu 2020-12-17 05:45:22 UTC
                 RTC time: Sat 2020-12-19 05:42:18
                Time zone: Asia/Seoul (KST, +0900)
System clock synchronized: no
              NTP service: n/a
          RTC in local TZ: no
```
