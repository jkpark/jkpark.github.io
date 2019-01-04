---
layout: post
title: Ubuntu18.04 - 06 - Synergy
description: Linux <-> Windows 간 마우스 및 키보드 공유 소프트웨어
category: blog
tags: [Ubuntu, Ubuntu18.04, Synergy]
---

[Synergy](https://symless.com/synergy)는 마우스와 키보드를 공유해주는 소프트웨어이다. `Syngery`를 사용하여 윈도우PC에 연결되어 있는 마우스와 키보드를 우분투18.04에서도 사용하도록 만들 것이다.

# Install Syngery on Windows 7

>On 8 September 2014, the Synergy developers started charging a fee for distribution of pre-compiled binary files of Synergy on their website, while offering a free download for the source code. The developers claim only "0.002% of people were donating" to fund the development before charging.

윈도우에서 `Synergy`를 사용하려면 구매를 해야한다. 하지만 `Synergy`는 [GNU General Public License](https://en.wikipedia.org/wiki/GNU_General_Public_License)를 따르기 때문에 소스코드를 빌드하여 사용할 수 있다. 엄밀히 말하면 다른 누군가가 빌드를 하여 공유하여도 개발자가 재재를 못한다. 때문에, 인터넷 상에 빌드된 무료버전을 쉽게 구할 수 있다.

아자님의 블로그를 통해 빌드된 `Synergy`를 다운받았다. [이 곳](https://ommokazza.blogspot.com/2017/07/synergy_19.html)에서 인스톨러를 다운로드한다.

다운로드 후 설치를 한다. 설치 과정은 쉽기 때문에 생략한다.

## 서버로 설정

윈도우PC를 메인으로 하기 때문에 윈도우를 서버로 한다. 우분투를 서버로 하고 윈도우에서 작업할 경우 마우스 키보드 사용에 따른 네트워크 트래픽이 유발되기 때문에 자신의 메인PC를 서버로 하는 것이 좋다.

`Synergy`를 실행시켜 `편집 - 설정`을 아래 그림과 같이 설정 후 OK를 누른다.

![](/images/posts/install-ubuntu1804/synergy00.png)

다음으로 서버를 체크하고 서버설정 버튼을 누른다.

![](/images/posts/install-ubuntu1804/synergy01.png)

아래와 같이 화면 레이아웃을 구성한다.
나는 우분투PC의 모니터가 윈도우PC의 모니터의 왼쪽에 있기 때문에 왼쪽에 놓았다.
그리고 화면이름은 `cactus`로 지정하였다.
우분투에서도 마찬가지로 동일한 화면이름으로 설정해주어야 한다.

![](/images/posts/install-ubuntu1804/synergy02.png)

단축키 설정은 가끔 클라이언트에서 작업하다 대기화면으로 전환되면 마우스 키보드가 먹통이 되기 때문에 윈도우화면으로 되돌아올 수 있도록 단축키를 설정한다.

![](/images/posts/install-ubuntu1804/synergy03.png)

클립보드 등 옵션을 설정한다.

![](/images/posts/install-ubuntu1804/synergy04.png)

시작을 누른다.
아래 그림과 같이 `started server, waiting for clients` 중이다.

![](/images/posts/install-ubuntu1804/synergy05.png)

# Install Synergy on Ubuntu 18.04

우분투에서는 `Synergy` Pro 버전이 무료로 제공된다.

```
$ sudo apt-get install synergy
```

`Synergy`를 실행시킨다.

아래와 같은 에러가 뜨는 건 무시해도 된다.

![](/images/posts/install-ubuntu1804/synergy06.png)

클라이언트 선택

![](/images/posts/install-ubuntu1804/synergy07.png)

`Edit - Settings`를 누르고 아래과 같이 설정한다.
SSL을 사용안하므로 체크해제한다.

![](/images/posts/install-ubuntu1804/synergy08.png)

`Start`를 누르면 정상적으로 연결이 되었을 경우 아래와 같이 알림 창이 뜬다.

![](/images/posts/install-ubuntu1804/synergy09.png)


## Autostart Synergy on bootup

### 로그인 전

[참고](https://help.ubuntu.com/community/SynergyHowto)

### 로그인 후

`~/.config/autostart/01synergyc.desktop` 파일을 만들고 아래 내용을 입력한다. `[server address]`에는 서버의 주소를 입력한다.

```
[Desktop Entry]
Encoding=UTF-8
Exec=synergyc [server address]
Name=Synergyc
Comment=Starting Synergyc
Terminal=false
OnlyShowIn=GNOME
Type=Application
StartupNotify=false
X-GNOME-Autostart-enabled=true
```