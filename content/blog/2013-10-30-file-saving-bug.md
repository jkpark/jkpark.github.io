---
layout: post
title: android(java) 파일 저장 후 리부팅 시 저장안되어 있는 현상
description: android에서 FileOutputStream 의 write();,  flush(); 후 리부팅 시 간헐적으로 파일이 내용이 변경되지 않을 때.
category: blog
tags: [Android]
---

android에서 FileOutputStream 의 `write();`, `flush();` 했는데도 리부팅 시 간헐적으로 파일이 생성되지 않았거나 파일 내 데이터가 변하지 않을 때가 있다.

FileOutputStream 의 `flush();` 가 버퍼를 비우는 기능만 수행하고 실제 physical device에 데이터를 저장하지 않아 파일에 데이터를 쓰기까지의 딜레이가 생기는 것이다.

이 때 `fos.getFD().sync();`의 메서드로 파일시스템 내부의 캐쉬를 모두 physical device로 저장한다.

```java
FileOutputStream fos = this.openFileOutput(filename, Context.MODE_PRIVATE);
fos.write(obj.toString().getBytes());
fos.flush();
fos.getFD().sync();
fos.close();
```
