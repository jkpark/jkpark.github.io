---
layout: post
title: Ubuntu18.04 - 04 - samba
description:
category: blog
tags: [Ubuntu, Ubuntu18.04, samba]
---

# Install Samba

`apt-get install samba` 로 `samba`를 설치한다.

# Add User

```
 $ sudo smbpasswd -a jkpark
New SMB password:
Retype new SMB password:
Added user jkpark.
```

# configuration

`/etc/samba/smb.conf` 파일을 열어 공유할 디렉토리를 설정한다.
맨 아랫줄에 아래 내용을 추가한다. 해당 디렉토리는 `btrfs`의 스냅샷 기능으로 파일복원이 가능하다. 이를 위한 설정도 몇개 추가했다.

```
# Enable the workspace directory shares. This will share each
# user's home directory as \\server\username
[homes]
  comment = %S's workspace

# path to sharing workspace
  path = /cactus_ws1/%S

# shadow_copy2 offers a functionality similar to MS shadow copy services.
# shadow_copy2 relies on a filesystem snapshot implementation.
# Enable for using BTRFS snapshot services.
  vfs objects = shadow_copy2

# snapshot saved directory
  shadow:snapdir = /btrfsmnt/@snapshot

# snapshot saved format
  shadow:format = @cactus_ws1_%Y.%m.%d_%H.%M.%S

# show snapshot as sorted
  shadow:sort = desc

# show this directory to other user browse network
  browseable = no

  read only = yes
  valid users = %S
  write list = %S
  create mask = 0644
  directory mask = 0755
```

설정 후 `systemctl restart smbd`로 삼바 재실행한다.

# 방화벽 설정

`iptables`로 `samba`가 사용하는 포트를 open시킨다.

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 135 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 137 -j ACCEPT
$ sudo iptables -A INPUT -p udp -m udp --dport 138 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 139 -j ACCEPT
$ sudo iptables -A INPUT -p tcp -m tcp --dport 445 -j ACCEPT
```

저장 후 리로드

```
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

# Access from Windows OS

`\\server_name\username`으로 접근

# restore each files

[btrfs 스냅샷 설정](install_ubuntu1804-02-btrfs#automatically-snapshot-creation) 에서 주기적으로 스냅샷이 적용되고 있기 때문에 파일의 `속성 - 이전 버전` 을 누르면 아래 그림과 같이 스냆샷 목록이 나온다.

![](/images/posts/install-ubuntu1804/samba01.png)

다만, 파일명으로 스냅샷을 찾기 때문에 파일명을 수정하거나 파일을 이동했다면 스냅샷을 불러오지 못한다. 이 경우 정확한 파일명을 알고 있다면 동일한 이름의 파일을 만들어 복원을 할 수 있다. 정확한 파일명을 모른다면 리눅스 상의 스냅샷을 뒤져가며 찾는 방법이 있다.
