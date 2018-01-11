---
layout: post
title: 토렌트서버(Transmission-daemon) 구축 방법
description: 우분투16.04에 Transmission-daemon 설치 로그.
category: blog
tags: [Torrent, Ubuntu16.04]
---

# history

## 다운로드 디렉토리 생성

downloads, incomplete 디렉토리 생성
```bash
jkpark@cactus:/storages/storage1/public/torrent$ ls -al  
합계 16  
drwxrwxr-x  4 jkpark jkpark  4096 10월 29 01:26 .  
drwxrwxrwx 13 nobody nogroup 4096 12월 28 21:43 ..  
drwxrwxrwx 12 jkpark jkpark  4096 12월 28 12:51 downloads  
drwxrwxrwx  2 jkpark jkpark  4096 10월 29 01:26 incomplete
```

원래 목적은 incomplete 디렉토리에 토렌트용 HDD를 마운트하고 이 위치에 다운로드해서 저장소의 부담을 줄이고 싶었지만 토렌트용 HDD가 고장났는지 인식이 안된다.
그러므로 다운로드를 downloads 디렉토리에 하도록 설정했다.
```bash
jkpark@cactus:/storages/storage1/public/torrent$ sudo apt-get install transmission-daemon
다음 새 패키지를 설치할 것입니다:
  transmission-cli transmission-daemon
```

설정파일이 생성되도록 데몬을 한번 실행하고 중지한다.
```bash
jkpark@cactus:/storages/storage1/public/torrent$ sudo service transmission-daemon start
jkpark@cactus:/storages/storage1/public/torrent$ sudo service transmission-daemon stop
```

## 설정
캐쉬 사이즈를 높게 설정할수록 IO작업이 줄어든다. 너무 높게 설정하면 RAM에 부담이 되므로 적당히 설정하자.
```
"cache-size-mb": 256,
```

다운로드할 위치
```
"download-dir": "/storages/storage1/public/torrent/downloads",
```

한번에 몇개의 다운로드를 진행할 것인지
```
"download-queue-enabled": true,
"download-queue-size": 4,
```

앞에서 말했듯이 토렌트용HDD가 고장나서 incomplete 디렉토리는 비활성화 한다.
```
"incomplete-dir": "/sotrages/storage1/public/torrent/incomplete",
"incomplete-dir-enabled": false,
```

아무곳에서나 접속할 수 있게 white list 는 `false` 한다.
```
"rpc-password": "비밀번호",
"rpc-username": "아이디",
"rpc-whitelist-enabled": false,
```


`umask`의 초기 값은 18인데 삼바에서 파일 엑세스 시 오류나서 0으로 바꿈
```
"umask": 0,
```

기타 설정방법은 [transmission wifi](https://github.com/transmission/transmission/wiki/Editing-Configuration-Files) 을 보고 참고한다.

*로그인 없이 접속하고 싶으면 `"rpc-authentication-required"`를 `false`


## 실행
```bash
jkpark@cactus:/etc/transmission-daemon$ sudo service transmission-daemon start
```

인터넷 브라우저에서 확인한다.

![][1]

아이디/비밀번호 입력

![][2]

  
동작되는 것을 확인할 수 있다.

* 안드로이드용 앱도 있다.  

![][3]

  

[1]: https://1.bp.blogspot.com/-pvjlX4pLQ9w/WGSOprhD7yI/AAAAAAAAAoo/pn1VJCO6Cowut9QS3_eDGttgAuLfhJbaACLcB/s320/%25EC%25BA%25A1%25EC%25B2%2598.PNG
[2]: https://4.bp.blogspot.com/-WzklqTH4Xrw/WGSOpt1qkoI/AAAAAAAAAos/YOwGfZtZeUgfkk9OGTILO_9BV3JhmgBAQCLcB/s320/%25EC%25BA%25A1%25EC%25B2%25982.PNG
[3]: https://3.bp.blogspot.com/-AD3CAALaEJ0/WGSPkBgPm0I/AAAAAAAAAo0/kUCUzWhqB1kF1Tbjz23tllmlvegRadmnQCLcB/s320/%25EC%25BA%25A1%25EC%25B2%2598.PNG