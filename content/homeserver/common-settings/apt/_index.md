---
title: APT
description: 
date: 2020-12-08T12:44:40+09:00
draft: false
weight: 2
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---


# Proxy

우분투는 `apt` 명령어를 통해 소프트웨어 관리를 다룰 수 있다.

`/etc/apt/apt.conf` 파일을 연다. (파일이 없다면 생성될 것이다.)
```
$ sudo vi /etc/apt/apt.conf
```

아래 내용을 추가한다.

```
Acquire::http::Proxy "http://xx.xx.xx.xx:xx";
Acquire::https::Proxy "http://xx.xx.xx.xx:xx";
```

`:wq` 을 입력하고 저장 후 편집을 종료한다.

# 저장소 변경

우분투는 `/etc/apt/source.list` 및 `/etc/apt/source.list.d/` 에 포함된 파일에 지정된 저장소에서 패키지를 다운로드한다. (로컬 디렉토리나 install media을 통해서도 다운로드 할 수 있다.)

`/etc/apt/source.list` 파일 내용을 보면, 우분투 설치 시 국가를 미국으로 선택했기 때문에 미국 저장소(http://us.archive.ubuntu.com/ubuntu/)로 지정되어 있다. 미국에 있는 저장소를 통해 소프트웨어를 다운받는 것보다 한국에 있고 카카오에서 제공하는 저장소를 통해 다운받는 편이 훨씬 빠르기 때문에 카카오 저장소로 바꿔준다.

```
$ `apt update`sudo sed -i -e 's/us.archive.ubuntu.com/mirror.kakao.com/g' /etc/apt/sources.list
```

다시 `/etc/apt/source.list` 파일 내용을 보면, security 저장소를 제외하면 모든 저장소가 mirror.kakao.com 로 바뀐 것을 확인할 수 있다.

나는 security 저장소는 미러 저장소로 지정하는 것을 비추천한다. 미러 저장소는 어쨌거나 오리진과의 싱크를 맞추기까지의 시간이 걸리기 때문에 중요한 보안패치는 오리진에서 바로 하는 것을 추천한다.

# update

`apt update` 명령어는 사용 가능한 패키지들과 버전들의 **리스트**를 업데이트 하는 명령어다.

```
$ sudo apt update
```

# upgrade

`apt update`를 통히 최신 정보를 업데이트했다면, `apt upgrade`를 통해 패키지를 업그레이드한다.

`apt upgrade`는 현재 설치되어 있는 모든 패키지들의 최신버전을 설치한다.

```
$ sudo apt upgrade
```

# other commands...

## dist-upgrade

기본적으로 `apt upgrade`는 **현재 설치되어 있는 패키지들만** 최신버전으로 설치한다. 다시 말하면, 업그레이드할 패키지가 새로운 의존성이 생기거나 의존성이 삭제되었을 때는 업그레이드가 거절된다. 그래서 커널이나 GRUB 등 의존성이 높은 패키지는 `apt upgrade`만으로 업그레이드 되지 않는다.

이런 의존성 높은 패키지를도 업그레이드하고자 할 때, `apt dist-upgrade` 명령어를 사용한다.

## remove

패키지를 삭제할 때는 `apt remove`를 사용한다.

## purge

`apt remove`로 패키지를 삭제하더라도 user configuration 파일은 삭제되지 않을 수 있다. 이는 remove를 잘못했을 경우를 위해 패키지를 다시 설치할 경우를 대비한 것이다. 만약 user configuration 파일들도 전부 삭제하고 싶을 떄는 `apt purge` 명령어를 사용한다. (참고로 `apt purge`를 하더라도 home 디렉토리에 있는 설정 파일들은 삭제되지 않는다.)