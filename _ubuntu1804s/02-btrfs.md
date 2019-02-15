---
layout: post
title: 02 - btrfs
description: 파일 백업 및 복원을 위한 btrfs 파티션 설정방법
category: ubuntu1804
---

# workspace 용 1TB HDD를 btrfs로 관리

`btrfs`는 파일의 예전 상태로 되돌릴 수 있는 스냅샷 기능을 지원한다. `btrfs`로 포맷한 파티션에 서브볼륨으로 `@workspace`를 생성하고, `@workspace`를 `/workspace`로 마운트한다. 그리고 `@workspace`를 `@workspace20181201`라는 이름으로 스냅샷을 저장해두었다가 나중에 `@workspace`를 삭제하고 `@workspace20181201`를 `@workspace`로 이름만 바꿔서 리마운트하면 해당 스냅샷으로 되돌아간다. 주의할 점은 스냅샷을 `@workspace` 하위 위치에 저장하면 안된다. 하위 위치에 스냅샷을 저장할 경우 해당 스냅샷으로 되돌아가면 시스템이 꼬인다. 그러므로, 스냅샷은 서브볼륨과 동등한 위치에 관리해야 한다. 예를 들어 `btrfs`로 포맷한 파티션을 `/btrfsmnt`에 마운트 시켰다면 이 위치에 서브볼륨과 스냅샷이 있는 구조로 가야한다. 

## btrfs 파티션 만들기

`fdisk -l` 명령어로 현재 파티션을 볼 수 있다.

```
$ sudo fdisk -l
Disk /dev/sda: 238.5 GiB, 256060514304 bytes, 500118192 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa4dfedfa

Device     Boot Start       End   Sectors   Size Id Type
/dev/sda1  *     2048 500117503 500115456 238.5G 83 Linux


Disk /dev/sdb: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x88d7b0d9
```

비어있는 `/dev/sdb`에 `btrfs` 파티션을 만들자.

```
$ sudo fdisk -c /dev/sdb

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-1953525167, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-1953525167, default 1953525167):

Created a new partition 1 of type 'Linux' and of size 931.5 GiB.

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'.

Command (m for help): p
Disk /dev/sdb: 931.5 GiB, 1000204886016 bytes, 1953525168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x88d7b0d9

Device     Boot Start        End    Sectors   Size Id Type
/dev/sdb1        2048 1953525167 1953523120 931.5G 8e Linux LVM

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

파티션을 만들었다면 파티션 테이블 변경사항을 커널에 업데이트 해야한다.

```
$ sudo partprobe -s
/dev/sda: msdos partitions 1
/dev/sdb: msdos partitions 1
```

## 파일시스템 설정

```
$ sudo mkfs.btrfs /dev/sdb1
```

## 마운트

생성한 파티션을 `/btrfsmnt`에 마운트할 것이다. 이 디렉토리는 스냅샷과 서브볼륨을 관리한다.

```
$ sudo mkdir /btrfsmnt
$ sudo mount /dev/sdb1 /btrfsmnt
```

`df` 명령어로 마운트되었는지 확인할 수 있다.

```
$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sdb1      btrfs     932G   17M  930G   1% /btrfsmnt
```

## subvolume 설정

스냅샷으로 서브볼륨 전체를 복원할 수도 있고 서브볼륨에 있는 각각의 파일도 복원 가능하다.

서브볼륨으로 `@cactus_ws1`를 생성할 것이다. 서브볼륨이라는 것을 나타내기 위해 `@`을 붙였다. 

```
$ sudo btrfs subvolume create /btrfsmnt/@cactus_ws1
$ sudo btrfs subvolume list /btrfsmnt
ID 258 gen 9 top level 5 path @cactus_ws1
```
`/btrfsmnt`에 `@cacus_ws1`이 생겼는지 확인한다.

```
$ ls -l /btrfsmnt/
drwxr-xr-x 1 root root 0 Dec 21 01:15 @cactus_ws1
```

## 서브볼륨 마운트

마운트는 `/cactus_ws1`에 한다. 이 곳은 사용자들의 workspace가 될 것이며 스냅샷 관리를 할 것이다.

```
$ sudo mkdir /cactus_ws1
$ sudo mount -o subvol=@cactus_ws1 /dev/sdb1 /cactus_ws1
```

## Automatically Mounting 

위에서 설정한 마스터 볼륨과 서브볼륨을 부팅 시 자동으로 마운트되도록 설정한다.

먼저 disk의 UUID를 확인

```
$ ll /dev/disk/by-uuid/
lrwxrwxrwx 1 root root  10 Dec 21 01:02 36f90d4f-f1d3-42a9-ab04-ee7e9105fe62 -> ../../sdb1
```

`/etc/fstab`에 아래 내용 추가

```
UUID=36f90d4f-f1d3-42a9-ab04-ee7e9105fe62 /cactus_ws1    btrfs   defaults,subvol=@cactus_ws1        0       2
UUID=36f90d4f-f1d3-42a9-ab04-ee7e9105fe62 /btrfsmnt     btrfs   defaults        0       2
```

`reboot` 명령어를 통해 시스템을 재부팅하여 자동으로 마운트가 잘 되는지 확인한다.


## 스냅샷 디렉토리 설정

```
$ sudo btrfs subvolume create /btrfsmnt/@snapshot
```

`@snapshot`이 `@cactus_ws1`과 같은 `top level 5` 인지 확인한다.

```
$ sudo btrfs subvolume list /btrfsmnt
ID 258 gen 14 top level 5 path @cactus_ws1
ID 259 gen 15 top level 5 path @snapshot
```

## 스냅샷 생성

```
sudo btrfs subvolume snapshot /btrfsmnt/@cactus_ws1 /btrfsmnt/@snapshot/@cactus_ws1_`date +%Y.%m.%d_%H.%M.%S`
```

주의할 점은 `date +%Y.%m.%d_%H.%M.%S`은 반드시 이대로 작성해야 `samba`를 통해 윈도우OS에서 스냅샷을 인식하고 복원할 수 있다.

스냅샷이 생성되었는지 확인한다.

```
$ sudo btrfs subvolume list /btrfsmnt
ID 258 gen 16 top level 5 path @cactus_ws1
ID 259 gen 16 top level 5 path @snapshot
ID 260 gen 16 top level 259 path @snapshot/@cactus_ws1_2018.12.21_02.22.11
```

## Automatically Snapshot Creation 

스냅샷을 생성하는 스크립트를 작성 후 `crontab`에 등록한다. 나는 하루에 한 번 백업할 것이기 때문에 `/etc/cron.daily`에 파일을 넣었다.

`/etc/cron.daily/btrfs-snapshot` 파일을 만들어 아래 내용을 작성한다.

```
#!/bin/bash
btrfs subvolume snapshot /btrfsmnt/@cactus_ws1 /btrfsmnt/@snapshot/@cactus_ws1_`date +%Y.%m.%d_%H.%M.%S`
```

실행 권한도 추가한다.

```
$ sudo chmod +x /etc/cron.daily/btrfs-snapshot
```

이제 `crontab`에 의해 하루에 한 번 스냅샷이 생성될 것이다.

## 스냅샷 복원

### 서브볼륨 전체 복원

복원하기 전 현재 디렉토리를 `mv`로 백업시켜 놓는다.

```
$ sudo mv /btrfsmnt/@cactus_ws1 /btrfsmnt/@cactus_ws1_last
```

[자동 마운트](#automatically-mounting) 단계에서 설정한 것처럼 `@cactus_ws1`의 이름으로 서브 볼륨이 마운트되기 때문에 스냅샷을  `@cactus_ws1`로 변경시키고 재부팅 혹은 리마운트하면 해당 스냅샷으로 복원된다.

```
$ sudo mv /btrfsmnt/@snapshot/@cactus_ws1_2018.12.21_02.22.11 /btrfsmnt/@cactus_ws1
```

리마운트 혹은 재부팅한다.

```
$ sudo umount /cactus_ws1
$ sudo mount -o subvol=@cactus_ws1 /dev/sdb1 /cactus_ws1
```

### 개별 파일 복원

디렉토리 전체 복원과 같이 복원하고자하는 스냅샷안에 파일을 `cp` 하면 된다.

#### 윈도우OS에서 복원

`samba`로 윈도우OS에서 접근가능하도록 설정 한 후 파일을 복원시킬 수 있다.
자세한 내용은 [restore each files from SMB](04-samba#restore-each-files) 에서 다룬다.


## 스냅샷 삭제

스냅샷 복원 및 파일명 변경은 `mv`를 통해 쉽게 할 수 있지만 스냅샷을 삭제하는 것은 아래 명령어를 통해 해야한다.

```
$ sudo btrfs subvolume delete /btrfsmnt/@cactus_ws1_last
```

## 사용자별로 workspace 나누기

[Make Own Workspace](03-general-setting#make-own-workspace)을 참고하여 사용자가 추가될 때 자동으로 workspace가 생기도록 만들어준다.