---
layout: post
title: burp suite 정리
description:
category: blog
tags: [burpsuite, Windows7]
---
# 프록시 설정 방법

메뉴 - Proxy - Options 에 Proxy Listeners 는 브라우저에서 나오는 request를 listen 한다.

```
127.0.0.1:8080 으로 설정
```

메뉴 - User Options - Upstream Proxy Servers 를 설정하면 아래와 같은 flow로 패킷이 지나간다.
브라우져 -> 127.0.0.1:8080 프록시 -> Burp Suite에서 catch -> Upstream Proxy Servers

| Enabled | Dest Host | Proxy Host     | Proxy Port |
| ------- | --------- | -------------- | ---------- |
| v       | 10.*      |                |            |

## 프록시 설정 바로가기 exe

```
C:\Windows\System32\control.exe inetcpl.cpl,,4
```

바탕화면에 `exe` 파일로 바로가기로 만들고 오른쪽클릭-속성에서 바로 가기 키를 `Ctrl+Alt+L` 로 지정.

`Ctrl+Alt+L` 로 인터넷옵션-연결 탭을 바로 열 수 있다.

`L`로 프록시 설정 탭을 열고 `S` `X` 를 눌러서  자동스크립트 구성 사용과 프록시 서버 설정을 끄고 킬 수 있다.

# 크롬 프록시 플러그인

크롬은 인터넷옵션-프록시 설정에 따르지 않고 독립적으로 프록시를 설정할 수 있다.
이렇게 설정하면 익스플로러를 burp suite용으로 사용하고 동시에 크롬을 다른 업무에 사용할 수 있다.

http://blog.naver.com/PostView.nhn?blogId=hmw53&logNo=60181498130 참고하여 플러그인 설치하자.
