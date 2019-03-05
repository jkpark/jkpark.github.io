---
layout: post
title: 6장. 파일 공유 - Samba
description:
category: ubuntu1804
---

# Overview

SMB(Server Message Block)는 윈도우즈 네트워크 파일 시스템의 기본 프로토콜 명이다. Samba는 리눅스상에서 SMB(Server Message Block) 프로토콜을 구현한 소프트웨어이기 때문에 리눅스의 파일을 윈도우PC에서 읽고 쓸 수 있게 한다.

## 공유할 디렉토리 목록

SMB를 통해 공유할 디렉토리이다. 각 디렉토리 별로 용도에 맞게 권한 설정도 할 것이다. (비인증 사용자(guest) 접근은 default 값으로 no로 설정 되어있다.)

|디렉토리|용도|소유자|
|/mnt/hdd1_btrfs/@workspace|작업공간|jkpark|
|/mnt/hdd1_btrfs/@pictures|사진|jkpark|
|/mnt/hdd1_btrfs/@music|음악|jkpark|
|/mnt/hdd1_btrfs/@videos|동영상|jkpark|
|/mnt/hdd1_btrfs/@private|개인적인 데이터|jkpark|
|/mnt/hdd1_btrfs/@public|공개 데이터|root|

`@public`의 경우 사용자명의 서브 디렉토리를 만들 것이다(예를 들어 /mnt/hdd1_btrfs/@public/jkpark).

# Samba 설치

```
$ sudo apt-get update
$ sudo apt-get install samba
```

# Samba 사용자 등록

Samba를 사용할 사용자를 등록한다. 여기서 설정하는 비밀번호는 시스템 로그인 시 사용하는 계정의 비밀번호와 연관이 없기 때문에 원하는 비밀번호로 설정한다. 나는 편의를 위해 시스템과 동일한 비밀번호로 설정하였다.

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
# 2019-02-26 | jkpark | enable the workspace directory share.
[workspace]
  comment = JK Park's workspace
  path = /mnt/hdd1_btrfs/@workspace
  browseable = no
  valid users = jkpark
  write list = jkpark
  create mask = 0644
  directory mask = 0755

# shadow_copy2 offers a functionality similar to MS shadow copy services.
# shadow_copy2 relies on a filesystem snapshot implementation.
# Enable for using BTRFS snapshot services.
  vfs objects = shadow_copy2
  shadow:snapdir = /mnt/hdd1_btrfs/snapshots/workspace
  shadow:format = @workspace_%Y.%m.%d_%H.%M.%S
  shadow:sort = desc
```

항목을 살펴보면 다음과 같다.

|**workspace**|네트워크 상 이름|
|path = **/mnt/hdd1_btrfs/@workspace**|공유할 디렉토리 경로. /home/jkpark/workspace로 지정해도 무방|
|browseable = **no**|네트워크 상에서 보여줄지|
|valid users = **jkpark**|접근 가능한 사용자 목록. 띄어쓰기로 구분 혹은 @그룹 지정|
|write list = **jkpark**|쓰기 권한이 있는 사용자. 띄어쓰기로 구분 혹은 @그룹 지정|
|create mask = 0644|파일 생성시 권한|
|directory mask = 0755|디렉토리 생성시 권한|
|vfs objects = shadow_copy2|스냅샷 사용을 위한 설정|
|shadow:snapdir = /mnt/hdd1_btrfs/snapshots/**workspace**|스냅샷이 저장되어 있는 디렉토리|
|shadow:format = @**workspace**_%Y.%m.%d_%H.%M.%S|스냅샷의 파일 형식|
|shadow:sort = desc|스냅샷을 내림차순 정렬|

위 내용은 디렉토리 하나에 대한 설정이므로 굵은 글씨 부분을 알맞게 수정하여 총 6개의 공유 디렉토리를 추가한다. 


설정 후 `systemctl restart smbd`로 삼바 재실행한다.

# 방화벽 설정

`samba`가 사용하는 포트에 대한 접속은 허용해주어야 한다. 

> `-s 192.168.122.0/24`와 같이 source도 명시 할 수 있다.

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 135 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 137 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 138 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 139 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 445 -j ACCEPT
```

저장 후 리로드

```
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

# 윈도우PC에서 접속

윈도우PC에서 접속은 크게 두 가지가 있다. `네트워크 드라이브 연결`은 드라이브 문자(예를 들어 'C')를 할당되기 로컬 드라이브처럼 데이터 공유, 권한 등을 설정할 수 있으므로 데이터 관리에 이점이 있다. 네트워크 드라이브 연결은 내부망 연결로 제한되어 있다. (VPN을 구성한다면 외부 네트워크에도 연결할 수 있다.)

`네트워크 위치`은 방화벽에 제한받지 않는다. 외부망의 폴더나 쉐어 링크 포인트를 가볍고 편하게 사용할 수 있다.

## 네트워크 드라이브 연결 방법

파임 탐색기에서 `컴퓨터` 오른쪽 클릭 -> `네트워크 드라이브 연결`을 클릭한다.

![new network drive 1](/images/ubuntu1804/samba/networkdrive1.png)

폴더 항목에 접속할 위치를 입력한다. IP주소 혹은 도메인명을 입력한다. 내 홈서버의 도메인 명은 `cactus`이다. 뒤에 붙는 `workspace`는 smb.conf 파일에 입력한 `[workspace]`이다. IP주소으로 연결 시 `\\192.168.122.2\workspace`와 같이 입력하면 된다. `다른 자격 증명을 사용하여 연결`을 체크하고 `마침` 버튼을 누르면 아래와 같이 로그인 창이 뜬다.

![new network drive 2](/images/ubuntu1804/samba/networkdrive2.png)

Samba 사용자 등록 시 생성한 계정을 입력하고 확인은 누르면 드라이브로 연결된다.

## 네트워크 위치 추가 방법


파임 탐색기에서 `컴퓨터` 오른쪽 클릭 -> `네트워크 위치 추가`을 클릭한다.

![new network location 1](/images/ubuntu1804/samba/networklocation1.png)

네트워크 드라이브 연결과 마찬가지로 연결할 위치를 지정하고 `다음` 버튼을 누른다.

![new network location 2](/images/ubuntu1804/samba/networklocation2.png)

표시할 폴더 이름을 입력하고 `다음` 버튼을 누른다. 마지막 화면에서 `마침` 버튼을 누른다.

## 아래와 같이 네트워크 드라이브와 네트워크 위치가 추가되었다.

![new network location 3](/images/ubuntu1804/samba/networklocation3.png)

# restore each files

btrfs 스냅샷 설정을 통해 주기적으로 스냅샷이 생성되기 때문에 여러번 수정을 거친 파일의 `속성 - 이전 버전`을 누르면 아래 그림과 같이 복원 가능한 시점이 나올 것이다.

![sample to restore from windows file explorer](/images/ubuntu1804/btrfs/00.png)

다만, 파일명으로 스냅샷을 찾기 때문에 파일명을 수정하거나 파일을 이동했다면 스냅샷을 불러오지 못한다. 이 경우 정확한 파일명을 알고 있다면 동일한 이름의 파일을 만들어 복원을 시도 할 수 있다.
