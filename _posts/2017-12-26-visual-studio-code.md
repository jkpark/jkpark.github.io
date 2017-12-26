---
layout: post
title: VS code 설치 및 설정 가이드
description: Instruction of installing VS code on windows 7 and basic settings.
category: blog
tags: [VS Code]
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

![](/images/posts/visual-studio-code/proxy-setting.PNG)




## C/C++ Extension

![C and CPP Extension](/images/posts/visual-studio-code/c-extension.PNG)

See <https://code.visualstudio.com/docs/languages/cpp>.


### To enable code completion and navigation

When I installed `C/C++ Extension` and `MinGW`, I still got lightbulb appears.

To Fix errors, I put more include pathes to `includePath` :
```
{
    "name": "Win32",
    "includePath": [
        "${workspaceRoot}",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++/mingw32",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++",
        "C:/MinGW/lib/gcc/mingw32/6.3.0/include/c++/tr1"
    ],
    "defines": [
        "_DEBUG",
        "UNICODE"
    ],
    "intelliSenseMode": "msvc-x64",
    "browse": {
        "path": [
            "${workspaceRoot}"
        ],
        "limitSymbolsToIncludedHeaders": true,
        "databaseFilename": ""
    }
}
```

### tasks.json

```
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build solution",
            "type": "shell",
            "command": "c:/MinGW/bin/g++",
            "args": [
                "-g",
                "solution.c"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

### launch.json
```
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/a.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\mingw\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "build hello world"
        }
    ]
}
```

## Code runner

![code runner](/images/posts/visual-studio-code/code-runner.PNG)

`Ctrl+Alt+N` to run code.

output :

![output](/images/posts/visual-studio-code/output.PNG)