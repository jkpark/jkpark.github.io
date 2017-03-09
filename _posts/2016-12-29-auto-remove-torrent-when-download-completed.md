---
layout: post
title: 토렌트 다운로드 완료 시 리스트에서 자동삭제
description: 토렌트 다운로드 완료 시 리스트에서 자동삭제 방법
category: blog
tags: [Torrent, Ubuntu16.04]
---

`transmission-daemon` 은 작업 시 꼭 `service`를 `stop` 한 상태에서 해야 수정작업이 적용이 된다.
```bash
jkpark@cactus:/storages/storage1/public/torrent$ sudo service transmission-daemon stop
```

`/etc/transmission-daemon/settings.json` 파일을 열어 다음과 같이 수정한다.
```
"script-torrent-done-enabled": true,
"script-torrent-done-filename": "/storages/storage1/public/torrent/auto_delete.sh",
```


`/storages/storage1/public/torrent/auto_delete.sh` 파일을 만들고 아래 내용을 입력한다.
```bash
#!/bin/sh
# Transmission script to remove torrent from lists

# The file for logging events from this script
LOGFILE=/storages/storage1/public/torrent/auto_delete_log

# Remote login details.
TR_HOST="9091 --auth=아이디:비밀번호"

echo "`date '+%Y-%m-%d %H:%M:%S'`  removed from list : $TR_TORRENT_NAME" >> $LOGFILE
transmission-remote $TR_HOST -t $TR_TORRENT_ID --remove
```

저장 후 권한설정
```bash
jkpark@cactus:/storages/storage1/public/torrent$ chmod 777 auto_delete.sh
jkpark@cactus:/storages/storage1/public/torrent$ sudo chown debian-transmission:debian-transmission auto_delete.sh
```

데몬 시작
```bash
jkpark@cactus:/storages/storage1/public/torrent$ sudo service transmission-daemon start
```


다운로드가 완료되면 리스트에서 토렌트가 제거되고 다음과 같이 로그가 남는다.
```bash
jkpark@cactus:/storages/storage1/public/torrent$ cat auto_delete_log
2016-12-29 22:05:48  removed from list : 제거된토렌트명
jkpark@cactus:/storages/storage1/public/torrent$
```