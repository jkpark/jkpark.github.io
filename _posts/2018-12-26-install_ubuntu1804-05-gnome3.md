---
layout: post
title: Ubuntu18.04 - 05 - Desktop Environment
description:
category: blog
tags: [Ubuntu, Ubuntu18.04, gnome3, chrome]
---

# Install Gnome 3 Desktop Environment

GNOME은 무료 오픈소스이며 Gnome3는 Ubuntu 18.04 desktop 의 공식적인 [Desktop_environment](https://en.wikipedia.org/wiki/Desktop_environment) 이다. Ubuntu 18.04 desktop 에 설치되는 Gnome의 경우 firefox, totem, libreOffice 같은 불필요한 어플리케이션들이 설치되고 Unity desktop과 비슷하게 보이기 위해서 변형된 Gnome이다. 나는 Ubuntu 18.04의 GUI는 체험해보기 위해 desktop버전과 동일한 패키지를 설치할 것이다. 다만 `--no-install-recommends` 옵션으로 libreoffice, firefox , thunderbird 같은 추천 소프트웨어를 제외시킬 것이다.

```
$ sudo apt-get install --no-install-recommends ubuntu-desktop
```

`reboot`을 하면 GUI 로그인창이 뜬다.

![](/images/posts/install-ubuntu1804/ubuntu_login_screen.png)

# Language and Keyboard Layout

## Install nimf
현재 내 시스템에는 `input method system`이 `XIM` 밖에 설치되어 있지 않다. 우분투 데스크탑 버전의 기본 입력기는 `ibus`이다. 하지만 `ibus`는 여전히 한글입력 시 버그가 존재한다. 이번 기회에 `nimf`를 설치할 것이다.

`nimf` PPA를 등록한다. ([이 PPA가 언제까지 유지될지 모르겠다.](https://cogniti-works.blogspot.com/2018/07/nimf-14.html))

```
$ sudo apt-add-repository ppa:hodong/nimf
$ sudo apt update
```

>만약 PPA가 등록이 안된다면 수동으로 등록해준다. `nimf`의 PPA는 [이 곳](https://launchpad.net/~hodong/+archive/ubuntu/nimf)에서 확인할 수 있다. 설치방법은 [이 곳](https://help.launchpad.net/Packaging/PPA/InstallingSoftware)을 참고하였다.
>
>`/etc/apt/sources.list.d/nimf.list` 파일을 만들고 아래 내용을 추가한다.
>
>```
>deb http://ppa.launchpad.net/hodong/nimf/ubuntu bionic main
>deb-src http://ppa.launchpad.net/hodong/nimf/ubuntu bionic main
>```
>
>키를 등록한다.
>
>```
>$ gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 91F9381B
>gpg: /home/jkpark/.gnupg/trustdb.gpg: trustdb created
>gpg: key C290E34891F9381B: public key "Launchpad PPA for Hodong Kim" imported
>gpg: Total number processed: 1
>gpg:               imported: 1
>$ gpg -a --export 91F9381B | sudo apt-key add -
>```
>
>`apt-get update`를 통해 PPA가 정상적으로 작동하는지 확인한다.
>
>```
>$ sudo apt-get update
>Get:2 http://ppa.launchpad.net/hodong/nimf/ubuntu bionic InRelease [15.9 kB]
>Get:4 http://ppa.launchpad.net/hodong/nimf/ubuntu bionic/main Sources [820 B]
>Get:5 http://ppa.launchpad.net/hodong/nimf/ubuntu bionic/main amd64 Packages [1,660 B]
>Get:7 http://ppa.launchpad.net/hodong/nimf/ubuntu bionic/main i386 Packages [1,660 B]
>Get:9 http://ppa.launchpad.net/hodong/nimf/ubuntu bionic/main Translation-en [812 B]
>Fetched 426 kB in 4s (114 kB/s)
>Reading package lists... Done
>```

`nimf`를 설치한다.

```
$ sudo apt install nimf nimf-libhangul
```

## Language Setting

`Language Support`를 실행시킨다.

![](/images/posts/install-ubuntu1804/language01.png)

`Install/Remove Languages`버튼을 눌러 `Korean`을 설치한다.

![](/images/posts/install-ubuntu1804/language02.png)

인터넷 검색 시 영어 자료가 훨씬 많이 나오기 때문에 한국어로 변경하지는 않는다.
다만 Regional Formats는 한국어로 설정해주었다. Keyboard input method system 도 `nimf`로 설정한다.

![](/images/posts/install-ubuntu1804/language03.png)
![](/images/posts/install-ubuntu1804/language04.png)

`Setting` 을 실행시켜 `Region & Language` 탭에서 `Input Sources`에 `+`를 눌러 `Korean(101/104 key compatible`을 추가하고 `English(US)`는 `-`버튼으로 지운다.

![](/images/posts/install-ubuntu1804/language05.png)

터미널을 열어 `gnome-session-quit`을 실행시켜 로그오프한 후 다시 로그인한다.

- - -
# Install Chrome browser

인터넷 접속을 위해 브라우저를 설치할 것이다. 오랫동안 크롬을 써왔기 때문에 크롬을 설치한다. 소프트웨어를 PPA를 통해 설치하면 `apt-get`으로 update를 할 수 있기 때문에 크롬도 PPA를 등록하여 `apt-get install`할 것이다.

```
$ wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
$ sudo sh -c 'echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list'
$ sudo apt-get update
$ sudo apt-get install google-chrome-stable
```

## Import Certification

`Settings - Manage certificates` 메뉴를 찾아 인증서를 등록한다. 인증서는 `/usr/share/ca-certificates/samsung/samsung.crt` 이다.

## Proxy

설정에서 프록시 설정도 해준다.
- - -

# Gnome Shell Extentions

Gnome Shell은 인터페이스를 변경할 수 있도록 Extentions 기능을 제공한다. Extentions을 통해 테마 설정 및 위젯등을 설치할 수 있다. 

![](/images/posts/install-ubuntu1804/gnome-result.png)

user theme extentions 을 설치하는 방법은 다양하다. 나는 크롬을 통해 설치할 것이다. 크롬에서 Gnome Shell을 연결하기 위해 `chrome-gnome-shell`을 설치한다.

https://extensions.gnome.org/ 에 접속하여 원하는 extention을 설치한다.
[이 곳을 참고](http://ubuntuhandbook.org/index.php/2017/05/enable-shell-theme-in-gnome-tweak-tool-in-ubuntu/)

설치목록:
- [User Themes](https://extensions.gnome.org/extension/19/user-themes/)
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)
- [Bing Wallpaper Changer](https://extensions.gnome.org/extension/1262/bing-wallpaper-changer/)



## Themes
테마 설정을 위해 `tweak`을 설치한다.

```
$ sudo apt-get install gnome-tweak-tool
```

www.gnome-look.org 에서 [Flat Remix GNOME theme](https://www.gnome-look.org/p/1013030/) 를 다운받았다.

`tweak`을 실행시켜 `Appearance` 메뉴의 `Theme`를 바꿔보자.

![](/images/posts/install-ubuntu1804/gnome-theme.png)



