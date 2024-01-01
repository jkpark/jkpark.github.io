---
layout: post
title: Windows 10 기본 설정
description: Windows 10 포맷 후 최적화 설정 방법
category: blog
tags: [Windows10]
---

# Windows 10 포맷 후 최적화 설정 방법

## One Drive 제거

한번에 제거할 수 있는 커맨드
[Github 페이지](https://github.com/tomchappelow/onedrive-uninstaller/blob/master/OneDrive-Uninstaller.cmd)

```bash
@echo off
cls

set x86="%SYSTEMROOT%\System32\OneDriveSetup.exe"
set x64="%SYSTEMROOT%\SysWOW64\OneDriveSetup.exe"

echo Closing the OneDrive process.
echo.
taskkill /f /im OneDrive.exe > NUL 2>&1
ping 127.0.0.1 -n 5 > NUL 2>&1

echo Uninstalling OneDrive...
echo.
if exist %x64% (
    %x64% /uninstall
    ) else (
      %x86% /uninstall
      )
  ping 127.0.0.1 -n 5 > NUL 2>&1

  echo Removing OneDrive leftovers...
  echo.
  rd "%USERPROFILE%\OneDrive" /Q /S > NUL 2>&1
  rd "C:\OneDriveTemp" /Q /S > NUL 2>&1
  rd "%LOCALAPPDATA%\Microsoft\OneDrive" /Q /S > NUL 2>&1
  rd "%PROGRAMDATA%\Microsoft OneDrive" /Q /S > NUL 2>&1

  echo Removing OneDrive from the Windows Explorer Side Panel...
  echo.
  REG DELETE "HKEY_CLASSES_ROOT\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f > NUL 2>&1
  REG DELETE "HKEY_CLASSES_ROOT\Wow6432Node\CLSID\{018D5C66-4533-4307-9B53-224DE2ED1FE6}" /f > NUL 2>&1

  pause
```

## Capslock to Control

아래 내용을 레지스트리에 등록

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout]
"Scancode Map"=hex:00,00,00,00,00,00,00,00,02,00,00,00,1d,00,3a,00,00,00,00,00
```

## 알 수 없는 장치 드라이버 ACPI\\INT33A0

Intel Smmart Connect Driver 를 설치하면 된다.

<https://downloadcenter.intel.com/download/23109/Intel-Smart-Connect-Technology>

<http://poeta.tistory.com/206> 참고

## Disable Cortana via Registry

Navigate to the following key

```
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Windows Search
```

만약 Windows 까지 갔는데 [Windows Search] 없다면 키 생성

![](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile9.uf.tistory.com%2Fimage%2F2271E13657F1276F135DE1)

Windows Search 폴더에서 [새로만들기 > DWORD(32비트)] -> 이름을 "AllowCortana" 라고 입력. 값은 "0"
![](http://img1.daumcdn.net/thumb/R1920x0/?fname=http%3A%2F%2Fcfile4.uf.tistory.com%2Fimage%2F236F273957F12872068A8C)

<http://mastmanban.tistory.com/943> 참고.

## Install Git

Download Git from <https://git-scm.com/downloads>.

## Install gVim

Download gVim from <https://vim.sourceforge.io/download.php#pc>.

and See <http://www.indiecoders.org/blog/2017/01/share-vimrc-file> for setting Plugins.

Registry for adding "Open with gVim" menu on Windows Explorer

```
Windows Registry Editor Version 5.00

[HKEY_CLASSES_ROOT\*\Shell\Vim]
@="Edit with &Vim"
"Icon"="\"C:\\Program Files (x86)\\Vim\\vim80\\gvim.exe\""

[HKEY_CLASSES_ROOT\*\Shell\Vim\command]
@="\"C:\\Program Files (x86)\\Vim\\vim80\\gvim.exe\" \"%1\""

[HKEY_CLASSES_ROOT\directory\background\shell\Vim]
@="Open with &Vim"
"Icon"="\"C:\\Program Files (x86)\\Vim\\vim80\\gvim.exe\""

[HKEY_CLASSES_ROOT\directory\background\shell\Vim\command]
@="\"C:\\Program Files (x86)\\Vim\\vim80\\gvim.exe\"
```
