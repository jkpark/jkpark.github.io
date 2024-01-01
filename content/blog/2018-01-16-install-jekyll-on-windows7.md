---
layout: post
title: Install Jekyll on Windows 7
description:
category: blog
tags: [Jekyll, Windows7]
---

이 포스트는 http://jekyll-windows.juthilo.com/ 를 참고 하였습니다.

# Install Ruby and Ruby DevKit

[Ruby Download Page](https://rubyinstaller.org/downloads/)

다운로드 ruby v2.3.3 and devKit.

```
rubyinstaller-2.3.3-x64.exe
DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe
```

> Jekyll has some dependencies which, out of the box, only provide raw source code. To make them into fully functional executables, you’ll probably need to install the Development Kit.

The download is a self-extracting archive. extract to `C:\RubyDevKit`

cmd 창에서

```
cd C:\RubyDevKit
ruby dk.rb init
```

init 후 `C:\RubyDevKit\config.yml` 파일에 아래 내용을 삽입한다.

```
---
 - C:/Ruby23-x64
```

install the devKit

```
ruby dk.rb install
```

# Gem Proxy Setting and strict ssl false

`~/.gemrc` 파일을 열어 아래 내용처럼 수정한다.

```
---
:backtrace: false
:bulk_threshold: 1000
:sources:
- https://rubygems.org/
:update_sources: true
:verbose: true
:ssl_verify_mode: 0
benchmark: false
gem: "--no-ri --no-rdoc --no-document --suggestions"
http-proxy: http://address:port
```

https proxy는 윈도우 환경변수에 등록시켜 준다.

```
set HTTPS_PROXY=https://address:port
```

# Gem update and clean up

```
gem update && gem cleanup
```

It is generally encouraged to keep your Gems updated

# Install jekyll

> **Compatibility**

> The latest version of Jekyll at the time of writing is v2.4.0, which is compatible with Windows. Most of the previous versions are, too. Do not attempt to install Jekyll v1.4.3, though, which is known to be incompatible with Windows.

> Check back here when a new version of Jekyll is released to find out if it remains compatible with Windows.

> http://jekyll-windows.juthilo.com/2-jekyll-gem/

```
gem install jekyll
```

```
$ jekyll -v
jekyll 3.7.0
```

## dependencies

```
gem install jekyll-sitemap
gem install jekyll-feed
gem install jekyll-paginate
```
