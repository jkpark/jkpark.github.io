---
layout: post
title: Resilio를 이용한 동기화
description: Resilio(구버전 btsync, BitTorrent Sync)를 이용한 안드로이드, 아이폰, 서버, 데스크탑 동기화 작업 내역
category: blog
tags: [Resilio, FileSync, Ubuntu16.04]
---

스마트폰에서 찍은 사진을 서버에 백업하고
다른 스마트폰과 공유하기 위해 동기화 프로그램이 필요하였다.
구글 드라이브같은 클라우드 서비스를 이용하면 간단하게 동기화를 할 수 있지만,
3자에게 정보를 공유해야하고 용량 제한, 업로드 속도 등 여러가지 제약이 많다.

Resilio Sync는 device to device 동기화이기 때문에 이번 기회에 사용해보고 싶었다.

![](https://4.bp.blogspot.com/-svFi0Ikw0Dg/WCviIDzsnzI/AAAAAAAAAkQ/EZ6VpP4ca2segoKl1S1VpLExJDlyXDpcQCEw/s320/%25EC%25BA%25A1%25EC%25B2%2598.PNG)


다른 싱크프로그램도 많지만 아이폰, 안드로이드 뿐만 아니라 리눅스, 윈도우 모든 OS에서 동작하는 싱크프로그램이 필요하기 때문에 Resilio를 택했다.

또한 Resilio Sync는 bitTorrent Sync 의 새로운 버전이고 토렌트방식으로 동기화를 한다.


이 포스트를 통해 다음을 진행할 것이다.

1. Ubuntu16.04에 Resilio 설치
2. Windows 10 desktop 에 Resilio 설치
3. 우분투와 윈도우간의 동기화 테스트
4. 안드로이드폰에 Resilio 설치 및 동기화 테스트
5. 아이폰에 Resilio 설치, 테스트

<br>

#### 1. Ubuntu 16.04에 Resilio 설치
이 스텝은 https://help.getsync.com/hc/en-us/articles/206178924 의 가이드를 참고 하였다.

resilio-sync.list 파일 생성 및 내용 추가

`/etc/apt/sources.list.d/resilio-sync.list` 파일을 열어 아래 내용추가한다.
```
deb http://linux-packages.resilio.com/resilio-sync/deb resilio-sync non-free
```


공개키 추가
```bash
jkpark@cactus:~$ wget -qO - https://linux-packages.resilio.com/resilio-sync/key.asc | sudo apt-key add -
OK
```

설치
```bash
jkpark@cactus:~$ sudo apt-get update
jkpark@cactus:~$ sudo apt-get install resilio-sync
```
설정
```bash
jkpark@cactus:~$ sudo systemctl enable resilio-sync
Synchronizing state of resilio-sync.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install enable resilio-sync
```

`/etc/resilio-sync/config.json` 파일을 열어

![](https://2.bp.blogspot.com/-ZrnOLvA2UjU/WCvnrMOPMlI/AAAAAAAAAks/Ncv_wdGj5lskBVV9W0Mkckka9HrPtrKXQCLcB/s500/%25EC%25BA%25A1%25EC%25B2%25984.PNG)

위 그림과 같이 `"listen"` 부분을 자신의 아이피로 설정


실행
```bash
jkpark@cactus:~$ sudo systemctl start resilio-sync
```

브라우저를 통해 서비스가 동작 중인것을 확인할 수 있다.

![](https://1.bp.blogspot.com/--WTOJ9O1YcE/WCvn2iTX7GI/AAAAAAAAAkw/WqTsp2tcSJkgzfEhpB9pD4O5PigjqoGDgCLcB/s500/%25EC%25BA%25A1%25EC%25B2%25985.PNG)

setting - preferences에서 언어를 한국어로 변경하였다.

![](https://4.bp.blogspot.com/--_Qe39e4Pns/WCvqR-541gI/AAAAAAAAAlQ/u1-0E27CQlQ2xMrOHr6wt8z4NDetHnDWwCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259810.PNG)

![](https://2.bp.blogspot.com/-bO-urYyvhDA/WCvqR7edCSI/AAAAAAAAAlU/J-jkTxAWqLExegF2OiAAy1SUG6lvQBJzwCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259811.PNG)


서비스 재시작하면 변경된 설정이 적용된다.
```bash
jkpark@cactus:~$ sudo systemctl restart resilio-sync
```


#### 2. Windows 10에 Resilio 설치
https://www.resilio.com/individuals/ 페이지에서 free download 를 눌러 설치파일을 다운로드한다.

설치방법은 간단하므로 생략한다.

![](https://1.bp.blogspot.com/-FF_N0AQGGJM/WCvo4THGOrI/AAAAAAAAAlA/dopoByyrylUiMxs591N9WwOkDZWAj97dACLcB/s500/%25EC%25BA%25A1%25EC%25B2%25986.PNG)

![](https://4.bp.blogspot.com/-laRZDAUXJEM/WCvo4eX6e0I/AAAAAAAAAk4/vpc0ACFwrZMOG2A0U9M2nBpzKd6f0tUawCLcB/s500/%25EC%25BA%25A1%25EC%25B2%25987.PNG)

![](https://1.bp.blogspot.com/-FySFHxhwMCM/WCvo4YHynkI/AAAAAAAAAk8/bDGoXpdCTjou1ICu8J87HafrlTkOjngpwCLcB/s500/%25EC%25BA%25A1%25EC%25B2%25988.PNG)




#### 3. 우분투와 윈도우간의 동기화 테스트

1. 우분투에서 폴더 추가 버튼을 눌러 동기화할 폴더를 선택후 열기버튼 클릭

![](https://4.bp.blogspot.com/-uCjhujHCziw/WCvrdAxA_kI/AAAAAAAAAls/FiZUJ1lM8gEI1Ba9hIxjKZpzuOYp-Hv_gCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259813.PNG)

2. 알맞게 설정 후 X버튼

![](https://1.bp.blogspot.com/-bjF-_84F6YI/WCvrdOp7oMI/AAAAAAAAAlw/p8icn-8DawI80DUou2LosbrnJLOw3wHTgCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259814.PNG)

3. 읽기 및 쓰기 키를 복사

![](https://3.bp.blogspot.com/-QLYTcUHAD0k/WCvucwrRC3I/AAAAAAAAAmQ/BjJWRiyhfcgDMSNqdDOxf81bruFyOXxJQCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259816.PNG)

4. 윈도우에서 키 또는 링크를 입력 버튼 클릭

![](https://1.bp.blogspot.com/-52gPM2U_9zY/WCvvJbQag_I/AAAAAAAAAmU/8RD6xRNgtyc0kzY1TnaBUESNxE7XUZaXACLcB/s500/%25EC%25BA%25A1%25EC%25B2%259817.PNG)


5. 복사한 키 붙여넣고 다음 클릭

![](https://3.bp.blogspot.com/-fh05rZYjcqc/WCvvJZx4bJI/AAAAAAAAAmY/JN8RKQtjNt4cw4JBBzxoyMdrEKIM7xHAgCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259818.PNG)

폴더 선택을 하면 다음과 같이 온라인 상태인 피어에 1of 1이라고 뜬다.
우분투와 윈도우가 동기화 되었다.

![](https://4.bp.blogspot.com/-LL-fJCBwBAY/WCvvJR3NPpI/AAAAAAAAAmc/4kY1dfKa9TclqDTx-PQ_cwO3OuPK2fswACLcB/s500/%25EC%25BA%25A1%25EC%25B2%259819.PNG)

아무 파일이나 동기화폴더인 sync_test_window 에 넣어보니
우분투의 test 폴더에 파일이 생성되었다.


#### 4. 안드로이드폰에 Resilio 설치 및 동기화 테스트

테스트한 기종은 V20 이다.

설치 후 테스트를 할 목적이었지만 방법이 어렵지 않기 때문에
주목적이었던 사진 폴더 동기화를 바로 진행하였다.


설치

![](https://3.bp.blogspot.com/-rBlhHMA2woI/WCwiSKhadyI/AAAAAAAAAm4/KJiqvas5Bj8rpnH8QNQeSny8nsDka1oHwCLcB/s500/device-2016-11-16-175620.png)

이름 설정

![](https://4.bp.blogspot.com/-vpEwy_PxrQs/WCwiSNKkLdI/AAAAAAAAAm8/nyWO88lApRIMNsJyl39uQU2Sel7tTLKXwCLcB/s500/device-2016-11-16-175844.png)

폴더 만들기 클릭

![](https://1.bp.blogspot.com/-40iWT6vgO4I/WCwiSChl6eI/AAAAAAAAAm0/PLeun1SULKwpMKfdcDpZ443bSZAA-DJPQCLcB/s500/device-2016-11-16-180222.png)

위치 지정 (카메라 기본 위치 선택함)

![](https://4.bp.blogspot.com/-wWDUQkwbd3M/WCwiSolCoyI/AAAAAAAAAnA/6JijjEdsyWo6A341A0GeZ34xs4U5_T5CgCLcB/s500/device-2016-11-16-180316.png)

완료.
우측의 info 아이콘 클릭

![](https://1.bp.blogspot.com/-7am8kgBsmc0/WCwiSiI_u5I/AAAAAAAAAnE/UCacEJDu_CEBfZcEaxLtvc2cb0zCKgUNgCLcB/s500/device-2016-11-16-180328.png)

권한 : 읽기 및 쓰기 (서버에서 삭제/추가 가능하도록)
링크 만료 : 절대 안함

![](https://1.bp.blogspot.com/-8gHtSGyy-cc/WCwiS6cPgGI/AAAAAAAAAnI/hbLEWL887DQdDYGPpbEFBGl1AF69mOhKwCLcB/s500/device-2016-11-16-180357.png)

선택적 동기화 해제 후 공유하기 클릭 (공유링크를 카카오톡으로 보냄)

![](https://2.bp.blogspot.com/-rYlSdEN1nlA/WCwiTBw7T2I/AAAAAAAAAnU/1Lmlu7wNaewurUiWTHX1Cu5ahl62fSDpQCLcB/s500/device-2016-11-16-180412.png)


윈도우로 돌아와서 

폴더 추가 옆 + 버튼 클릭 - 키 또는 링크를 입력 클릭
공유 링크 붙여넣기

![](https://1.bp.blogspot.com/-Ju0pqCI8ddM/WCwiUNylkzI/AAAAAAAAAng/SKzaalTgxi4htwzDbz_EKiBx-24btIWvwCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259820.PNG)

저장 위치 지정

![](https://2.bp.blogspot.com/-TOmxuQHghUU/WCwiUUAF1xI/AAAAAAAAAnk/poBRpBEDiE8PW32qQMptWRzCM6zExET5wCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259821.PNG)

연결 버튼을 누르면 아래 그림과 같이 연결중... 에서 승인 대기 중으로 바뀐다.

![](https://2.bp.blogspot.com/-THqEnquc_xQ/WCwiUobdUOI/AAAAAAAAAns/rj6_QsmPDCk_zED1PdrBuYbobEW0h-f3ACLcB/s500/%25EC%25BA%25A1%25EC%25B2%259822.PNG)

![](https://1.bp.blogspot.com/-aYBAvnjoSgw/WCwiUWstCiI/AAAAAAAAAno/JWexDeepWSwlmPuQT8SsSBSQ-kFV9D0aACLcB/s500/%25EC%25BA%25A1%25EC%25B2%259823.PNG)

스마트폰으로 돌아와서

우측상단에 알림 아이콘 클릭

승인

![](https://4.bp.blogspot.com/-6pOetOs7cAE/WCwiTQT_L6I/AAAAAAAAAnM/lvjaMP2wbo061aljxddFiUWYKmyZdhdlACLcB/s500/device-2016-11-16-180643.png)

승인이 완료되면 동기화를 시작한다.

![](https://1.bp.blogspot.com/-qhCS8iaq-60/WCwiT15pcuI/AAAAAAAAAnc/RHykztW8IMcWJbTNCGhS8nFayKcsG0D_gCLcB/s500/device-2016-11-16-180744.png)

우측하단 ↑11.4 MB/s 는 현재 업로드 속도이다.

윈도우로 돌아와서
공유 지정 폴더를 보면 아래와 같이 사진이 동기화 되었다.

![](https://3.bp.blogspot.com/-1tCUR8HxW3c/WCwiU6yKb9I/AAAAAAAAAnw/W5qmfz4oAvwfVuqDQx0zOA-wAfEg3zJjwCLcB/s500/%25EC%25BA%25A1%25EC%25B2%259824.PNG)

물론 쓰기 권한이 있기 때문에 추가/수정/삭제가 가능하고

동기화 폴더에 변화가 있으면 자동으로 스마트폰에서 동기화를 시작한다.


#### 5: 아이폰에 Resilio 설치, 테스트

설치 방법 및 일반 폴더 동기화 방법은 안드로이드와 동일하다.

하지만 예상과는 달리 아이폰의 카메라 같은 경우 읽기모드만 지원하기때문에
서버에 백업은 했지만 서버에서 추가/수정/삭제가 불가능하다.
또한 동기화가 필요한 경우 어플을 실행해야하는 불편함이 있다.