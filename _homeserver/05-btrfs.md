---
layout: post
title: 5장. 데이터 백업 및 복원이 가능한 파일 시스템 - btrfs
category: homeserver
---

# Overview

2TB HDD에 사진, 공문서 같은 개인적인 데이터를 포함하여 각종 데이터를 저장할 것이기 때문에 데이터 백업과 복원은 상당히 중요하다. btrfs(Butter file system or b-tree file system)은 스냅샷을 이용한 데이터 백업 및 복원, 자체 압축, 서브볼륨 관리 등이 장점이다.

btrfs는 자신의 볼륨을 여러개의 서브볼륨 형태로 나눌 수 있기 때문에 디스크 전체를 btrfs 파티션으로 생성하고 서브볼륨 단위로 나눈다. 파티션의 경우 크기을 정해주어야 하는 것과 달리 btrfs의 서브볼륨은 용량에 제한이 없다. 또한, 각각의 서브볼륨을 스냅샷으로 저장해두면 언제든지 저장시점으로 되돌릴 수 있다. 스냅샷 이름의 형식을 윈도우 파일 버전 관리 이름 형식으로 마춰준다면 윈도우 파일 탐색기를 통해서도 복원이 가능하다.

![sample to restore from windows file explorer](/images/ubuntu1804/btrfs/00.png)
*SMB를 통해 윈도우 파일 탐색기에서도 홈 서버의 파일을 이전 버전으로 되돌릴 수 있다.*

복원시점인 스냅샷은 서브볼륨 단위로 생성하고 삭제할 수 있다. 스냅샷에는 해당 시점에 있는 모든 파일들이 담겨있어서 스냅샷을 생성해 두면 파일을 삭제하여도 언제든지 복원이 가능하다. 해당 스냅샷 안에 있는 파일을 현재 서브볼륨으로 복사하는 것으로도 개별 파일 복원이 가능하다.

어떤 파일을 완전히 삭제하기 위해선 서브볼륨과 서브볼륨으로 생성한 모든 스냅샷에서 해당 파일을 지워야 한다. 다시 말하면, 모든 스냅샷에서 해당 파일을 지우지 않는 이상 여전히 디스크의 용량을 차지하고 있게 된다. 주기적으로 생성되는 스냅샷은 디스크 용량이 부족하거나 정리가 필요할 때 과감하게 지울 수 있어야 한다. 서브볼륨을 적절히 나누어 스냅샷 삭제 시 다른 서브볼륨에 영향이 없도록 한다.

|서브볼륨|용도|스냅샷 생성 주기|
|workspace|작업공간|1일|
|cloud|클라우드의 저장소|월,금,토,일|
|private|개인문서 같은 중요 파일 보관|1일|
|pictures|사진|1일|
|music|음악|-|
|torrent|토렌트다운로드|-|
|public|그 외 모든 파일|1일|


# 파티션 생성

`fdisk -l` 명령어로 현재 파티션을 볼 수 있다.

```
$ sudo fdisk -l
...

Disk /dev/sdb: 1.8 TiB, 2000365289472 bytes, 3906963456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00021365
```

비어있는 `/dev/sdb`에 새로운 파티션을 생성하기 위해 `fdisk` 명령어를 이용한다.

```
$ sudo fdisk -c /dev/sdb

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): 
```

위와 같이 `Command (m for help):` 프롬프트가 떴으면 순서대로 진행한다.
```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-3906963455, default 2048): 
Last sector, +sectors or +size{K,M,G,T,P} (2048-3906963455, default 3906963455): 

Created a new partition 1 of type 'Linux' and of size 1.8 TiB.
```

파티션이 생성되었다. `P` 명령어로 파티션 정보를 출력할 수 있다.
```
Command (m for help): p
Disk /dev/sdb: 1.8 TiB, 2000365289472 bytes, 3906963456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00021365

Device     Boot Start        End    Sectors  Size Id Type
/dev/sdb1        2048 3906963455 3906961408  1.8T 83 Linux
```

파티션 타입을 변경하기 위해 `L`로 파티션 타입 리스트를 출력한다.
```
Command (m for help): l

 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt        

```

`LVM`으로 선택한다.
```
Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'.
```

변경사항을 저장한다.
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

파티션을 만들었다면 파티션 테이블 변경사항을 커널에 알려야 한다.

```
$ sudo partprobe -s
/dev/sda: gpt partitions 1 2 3
/dev/sdb: msdos partitions 1
```

`/dev/sdb` 디스크에 `/dev/sdb1` 파티션이 생성된 결과

```
$ sudo fdisk -l /dev/sdb
Disk /dev/sdb: 1.8 TiB, 2000365289472 bytes, 3906963456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00021365

Device     Boot Start        End    Sectors  Size Id Type
/dev/sdb1        2048 3906963455 3906961408  1.8T 8e Linux LVM
```

# 파일시스템 설정

`/dev/sdb`를 btrfs 파일시스템으로 만든다.

```
$ sudo mkfs.btrfs /dev/sdb1
btrfs-progs v4.15.1
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               5151998b-c75b-4554-87b2-9dd40349b890
Node size:          16384
Sector size:        4096
Filesystem size:    1.82TiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP               1.00GiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1     1.82TiB  /dev/sdb1
```

# 마운트

btrfs 최상위 볼륨 `/dev/sdb1`을 `/mnt/hdd1_btrfs`에 마운트한다.

```
$ sudo mkdir /mnt/hdd1_btrfs
$ sudo mount /dev/sdb1 /mnt/hdd1_btrfs
```

`df` 명령어로 마운트된 디스크 정보를 확인한다.

```
$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sdb1      btrfs     1.9T   17M  1.9T   1% /mnt/hdd1_btrfs
```

진행에 앞 서 `/mnt/hdd1_btrfs`로 이동한다.

```
$ cd /mnt/hdd1_btrfs
```

# 서브볼륨 생성

**생성할 서브볼륨 리스트**

|서브볼륨|용도|소유자|
|workspace|작업공간|jkpark|
|cloud|클라우드의 저장소|root(추후변경)|
|private|개인문서 같은 중요 파일 보관|jkpark|
|pictures|사진|jkpark|
|music|음악|jkpark|
|torrent|토렌트 다운로드|root(추후변경)|
|public|그 외 모든 파일|root|


서브볼륨들을 생성한다. 서브볼륨이라는 것을 나타내기 위해 `@`을 붙였다. 

```
$ sudo btrfs subvolume create @workspace
$ sudo btrfs subvolume create @cloud
$ sudo btrfs subvolume create @private
$ sudo btrfs subvolume create @pictures
$ sudo btrfs subvolume create @music
$ sudo btrfs subvolume create @torrent
$ sudo btrfs subvolume create @public
```

생성된 서브볼륨 리스트를 확인한다. 서브볼륨의 ID과 top level, 경로가 출력된다.

```
$ sudo btrfs subvolume list /mnt/hdd1_btrfs
ID 265 gen 19 top level 5 path @workspace
ID 266 gen 20 top level 5 path @cloud
ID 267 gen 21 top level 5 path @private
ID 268 gen 22 top level 5 path @pictures
ID 269 gen 23 top level 5 path @music
ID 270 gen 24 top level 5 path @torrent
ID 271 gen 25 top level 5 path @public
```

btrfs의 서브볼륨은 블록 디바이스가 아닌 독립적이고 마운트 가능한 POSIX 파일트리이다. 즉 마운트 시 마운트 지점의 소유자는 서브볼륨의 소유자가 된다. 생성할 서브볼륨 리스트에서 보았듯이 서브볼륨의 소유자도 변경한다.

```
$ sudo chown jkpark: @workspace/ @private/ @pictures/ @music/
```

`ll` 명령어로 파일의 소유자를 확인한다.

```
$ ll
total 20
drwxr-xr-x 1 root   root    108 Mar  7 20:12  ./
drwxr-xr-x 3 root   root   4096 Mar  7 19:50  ../
drwxr-xr-x 1 root   root      0 Mar  7 20:13 '@cloud'/
drwxr-xr-x 1 jkpark jkpark    0 Mar  7 20:14 '@music'/
drwxr-xr-x 1 jkpark jkpark    0 Mar  7 20:14 '@pictures'/
drwxr-xr-x 1 jkpark jkpark    0 Mar  7 20:14 '@private'/
drwxr-xr-x 1 root   root      0 Mar  7 20:14 '@public'/
drwxr-xr-x 1 root   root      0 Mar  7 20:14 '@torrent'/
drwxr-xr-x 1 jkpark jkpark    0 Mar  7 20:13 '@workspace'/
```

# 서브볼륨 마운트

서브볼륨 들을 원하는 위치에 마운트한다. 나는 아래 위치에 마운트 시킬 것이다.

|@workspace|/home/jkpark/workspace
|@private|/home/jkpark/private
|@pictures|/home/jkpark/Pictures
|@music|/home/jkpark/Music
|@torrent|/torrent
|@public|/public
|@cloud|/cloud

```
$ mkdir /home/jkpark/workspace /home/jkpark/private /home/jkpark/Pictures /home/jkpark/Music
$ sudo mkdir /torrent /public /cloud
```

`mount` 명령어로 마운트한다.

```
$ sudo mount -o subvol=@public /dev/sdb1 /public
```

마운트 해제는 `umount`명령어로 한다.

```
$ sudo umount /public
```

## 마운트 자동화

부팅 시 btrfs 최상위 볼륨과 서브볼륨들이 자동으로 마운트되도록 설정한다.

먼저 btrfs 볼륨의 UUID를 확인한다.

```
$ ll /dev/disk/by-uuid/
lrwxrwxrwx 1 root root  10 Mar  7 19:49 5151998b-c75b-4554-87b2-9dd40349b890 -> ../../sdb1
```

`5151998b-c75b-4554-87b2-9dd40349b890` 가 btrfs 볼륨 /dev/sdb1 의 UUID이다. 

마운트를 자동화하기 위한 설정 파일은 `/etc/fstab` 이다. 부팅 시 이 파일을 참고하여 디스크가 마운트 된다.

`/etc/fstab` 파일을 열어 아래 내용을 추가한다.

```
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /mnt/hdd1_btrfs    btrfs    defaults    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /home/jkpark/workspace    btrfs    defaults,subvol=@workspace    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /home/jkpark/private    btrfs    defaults,subvol=@private    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /home/jkpark/Pictures    btrfs    defaults,subvol=@pictures    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /home/jkpark/Music    btrfs    defaults,subvol=@music    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /torrent    btrfs    defaults,subvol=@torrent    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /public    btrfs    defaults,subvol=@public    0    2
UUID=5151998b-c75b-4554-87b2-9dd40349b890 /cloud    btrfs    defaults,subvol=@cloud    0    2
```

라인 맨 뒤 `0    2`는 각 각 dump과 pass 옵션이다.

>|dump|Enable or disable backing up of the device/partition (the command dump). This field is usually set to 0, which disables it.|
>|pass num|Controls the order in which fsck checks the device/partition for errors at boot time. The root device should be 1. Other partitions should be 2, or 0 to disable checking.|

`reboot` 명령어를 통해 시스템을 재부팅하여 마운트가 잘 되는지 확인한다.

시스템이 재부팅 되었다면 `findmnt` 명령어로 마운트된 btrfs을 확인한다.

```
$ findmnt -nt btrfs
/public                /dev/sdb1[/@public]    btrfs  rw,relatime,space_cache,subvolid=271,subvol=/@public
/mnt/hdd1_btrfs        /dev/sdb1              btrfs  rw,relatime,space_cache,subvolid=5,subvol=/
/home/jkpark/workspace /dev/sdb1[/@workspace] btrfs  rw,relatime,space_cache,subvolid=265,subvol=/@workspace
/home/jkpark/Music     /dev/sdb1[/@music]     btrfs  rw,relatime,space_cache,subvolid=269,subvol=/@music
/home/jkpark/private   /dev/sdb1[/@private]   btrfs  rw,relatime,space_cache,subvolid=267,subvol=/@private
/cloud                 /dev/sdb1[/@cloud]     btrfs  rw,relatime,space_cache,subvolid=266,subvol=/@cloud
/torrent               /dev/sdb1[/@torrent]   btrfs  rw,relatime,space_cache,subvolid=270,subvol=/@torrent
/home/jkpark/Pictures  /dev/sdb1[/@pictures]  btrfs  rw,relatime,space_cache,subvolid=268,subvol=/@pictures
```

# 스냅샷 디렉토리 설정

스냅샷 저장을 위한 디렉토리를 만든다.

```
$ sudo mkdir /mnt/hdd1_btrfs/snapshots
$ cd /mnt/hdd1_btrfs/snapshots
$ sudo mkdir workspace private pictures music torrent public cloud
```

스냅샷 저장 구조를 간략하면 아래와 같을 것이다.
```
 top-level                             (volume root directory, to be mounted at /mnt/hdd1_btrfs)
   +-- @workspace                      (subvolume root directory, to be mounted at /home/jkpark/workspace)
   +-- @public                         (subvolume root directory, to be mounted at /public)
   \-- snapshots                       (directory)
       \-- workspace                   (directory)
           +-- @workspace_2019-02-21   (root directory of snapshot of subvolume "@workspace")
           +-- @workspace_2019-02-22   (root directory of snapshot of subvolume "@workspace")
       \-- public                      (directory)
           +-- @public_2019-02-21      (root directory of snapshot of subvolume "@public")
           +-- @public_2019-02-22      (root directory of snapshot of subvolume "@public")
```

# 스냅샷 생성

`btrfs subvolume snapshot` 명령어로 스냅샷을 생성한다.

```
$ sudo btrfs subvolume snapshot /mnt/hdd1_btrfs/@workspace /mnt/hdd1_btrfs/snapshots/workspace/@workspace_`date +%Y.%m.%d_%H.%M.%S`
```

`date +%Y.%m.%d_%H.%M.%S` 는 `연.월.일_시.분.초` 형식으로 저장하라는 명령이다. 반드시 이대로 작성해야 `samba`를 통해 윈도우OS에서 스냅샷을 인식하고 복원할 수 있다.

스냅샷이 생성되었는지 확인한다.

```
$ sudo btrfs subvolume list /mnt/hdd1_btrfs
ID 265 gen 30 top level 5 path @workspace
ID 266 gen 20 top level 5 path @cloud
ID 267 gen 26 top level 5 path @private
ID 268 gen 26 top level 5 path @pictures
ID 269 gen 26 top level 5 path @music
ID 270 gen 24 top level 5 path @torrent
ID 271 gen 27 top level 5 path @public
ID 272 gen 30 top level 5 path snapshots/workspace/@workspace_2019.03.07_20.25.03
```

스냅샷 `snapshots/workspace/@workspace_2019.03.07_20.25.03`이 생성되었다. 

# 스냅샷 생성 스크립트

스냅샷을 생성하는 스크립트를 작성한다.

```
$ sudo vi /usr/local/etc/btrfs-snapshot.sh
```

아래 내용을 입력한다.

<script src="https://gist.github.com/jkpark/9ebf5d8f6260c8df04e4b521cacd02c3.js"></script>

`:wq` 을 입력하고 저장 후 편집을 종료한다.

실행권한을 추가한다.

```
$ sudo chmod +x /usr/local/etc/btrfs-snapshot.sh
```

`btrfs-snapshot.sh`에 스냅샷을 생성하고자하는 서브볼륨을 입력하면 스냅샷이 생성된다.

```
$ sudo /usr/local/etc/btrfs-snapshot.sh workspace
Create a snapshot of '/mnt/hdd1_btrfs/@workspace' in '/mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.03.07_20.26.53'
```

# 스냅샷이 주기적으로 실행되도록 crontab에 추가

위에서 생성한 스냅샷 생성 스크립트가 주기적으로 자동실행되도록 crontab에 등록한다.

**서브볼륨 별 스냅샷 주기**

|서브볼륨|용도|스냅샷 생성 주기|
|workspace|작업공간|1일|
|cloud|클라우드의 저장소|월,금,토,일|
|private|개인문서 같은 중요 파일 보관|1일|
|pictures|사진|1일|
|music|음악|-|
|torrent|토렌트 다운로드|-|
|public|그 외 모든 파일|1일|


`/etc/crontab` 파일을 연다.

```
$ sudo vi /etc/crontab
```

파일의 끝에 아래 내용을 추가하고 `:wq`로 저장 후 종료한다.

```
# 2019-03-07 | jkpark | added below 5 lines 
05 6    * * mon,fri,sat,sun   root    /usr/local/etc/btrfs-snapshot.sh cloud
06 6    * * *   root    /usr/local/etc/btrfs-snapshot.sh workspace
07 6    * * *   root    /usr/local/etc/btrfs-snapshot.sh private
08 6    * * *   root    /usr/local/etc/btrfs-snapshot.sh pictures 
09 6    * * *   root    /usr/local/etc/btrfs-snapshot.sh public 
```

`05 6    * * mon,fri,sat,sun root /usr/local/etc/btrfs-snapshot.sh cloud`을 살펴보면 매주 월,금,토,일요일 6시 5분에 root 권한으로 `/usr/local/etc/btrfs-snapshot.sh cloud`을 실행한다는 의미이다. 왼쪽부터 분, 시, 일, 월, 요일, 사용자, 명령어 순이다.

> `# 2019-03-07 | jkpark | added below 5 lines`는 주석이다. 이렇게 주석으로 수정내역을 적어 놓으면 서버 관리에 도움이 된다.

설정이 끝나면 cron 서비스를 재시작한다.

```
$ sudo systemctl restart cron
```

# 스냅샷으로 복원

현재 서브볼륨을 스냅샷으로 복원하기에 앞서 현재 디렉토리를 백업한다.

```
$ sudo mv /mnt/hdd1_btrfs/@workspace /mnt/hdd1_btrfs/@workspace_latest
```

복원할 스냅샷을 서브볼륨 이름으로 이동시킨다.

```
$ sudo mv /mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.02.26_01.34.00 /mnt/hdd1_btrfs/@workspace
```

리마운트한다.

```
$ sudo umount /home/jkpark/workspace
$ sudo mount -o subvol=@workspace /dev/vdb1 /home/jkpark/workspace
```

## 개별 파일 복원

파일 복원은 원하는 시점의 스냅샷에서 파일을 복사해오면 된다.

`@workspace`의 스냅샷 `@workspace_2019.02.26_18.02.53`에서 `helloworld` 파일을 복원할 경우

```
$ cp /mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.02.26_18.02.53/helloworld /home/jkpark/workspace/helloworld
```

*윈도우PC에서의 파일 복원은 **6장. 파일 공유 - Samba** 에서 다룰 것이다.*


## 스냅샷 삭제

스냅샷 복원은 `mv`를 통해 쉽게 할 수 있지만 스냅샷을 삭제하는 것은 삭제 명령어를 입력해야 한다.

```
$ sudo btrfs subvolume delete /mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.02.26_18.02.53
Delete subvolume (no-commit): '/mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.02.26_18.02.53'
```

`btrfs subvolume delete /mnt/hdd1_btrfs/snapshots/workspace/@workspace_2019.*` 같이 여러개의 스냅샷도 동시에 삭제할 수 있다.
