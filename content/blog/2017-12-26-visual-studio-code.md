---
layout: post
title: Setup Visual Studio Code
description: Instruction of installing VS code on windows 7.
category: blog
tags: [VS Code, Windows7]
english: true
---

Visual Studio Code is a lightweight but powerful source code editor which runs on your desktop and is available for Windows, macOS and Linux. It comes with built-in support for JavaScript, TypeScript and Node.js and has a rich ecosystem of extensions for other languages (such as C++, C#, Java, Python, PHP, Go) and runtimes (such as .NET and Unity). Begin your journey with VS Code with these [introductory videos](https://code.visualstudio.com/docs/introvideos/overview).


# Installation

1. Download the Visual Studio Code installer from <https://code.visualstudio.com/>.
2. Once it is downloaded, run the installer (VSCodeSetup-version.exe). This will only take a minute.
3. By default, VS Code is installed under C:\Program Files\Microsoft VS Code for a 64-bit machine.

## Proxy Setting

VS Code has exactly the same proxy server support.

`File > Preferences > Settings`, `Ctrl+Shift+P > Open User Settings` or shortcut (`Ctrl+,`) to Open Settings panel.

you can use the following command line arguments to change your proxy settings:

```
    "http.proxy": "http(s)://<ip address>:<port>"
```

