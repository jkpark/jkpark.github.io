---
title: Btrfs
description: 
oneliner: a file system based on the copy-on-write (COW) principle.
date: 2020-12-07T15:31:13+09:00
draft: false
weight: 0
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# Overview

256GB SSD에는 207GB만큼 btrfs를 할당하고, 2TB HDD에는 1TB만큼 btrfs를 할당할 것이다.

> ubuntu install 글에서는 128GB SSD로 설명했지만, 256GB SSD로 업그레이드하여 256GB로 설명한다.

btrfs는 자신의 볼륨을 여러개의 서브볼륨 형태로 나눌 수 있기 때문에 디스크 전체를 btrfs 파티션으로 생성하고 서브볼륨 단위로 나눈다. 파티션의 경우 크기을 정해주어야 하는 것과 달리 btrfs의 서브볼륨은 용량에 제한이 없다. 또한, 각각의 서브볼륨을 스냅샷으로 저장해두면 언제든지 저장시점으로 되돌릴 수 있다. 스냅샷 이름의 형식을 윈도우 파일 버전 관리 이름 형식으로 마춰준다면 윈도우 파일 탐색기를 통해서도 복원이 가능하다.

복원시점인 스냅샷은 서브볼륨 단위로 생성하고 삭제할 수 있다. 스냅샷에는 해당 시점에 있는 모든 파일들이 담겨있어서 스냅샷을 생성해 두면 파일을 삭제하여도 언제든지 복원이 가능하다. 해당 스냅샷 안에 있는 파일을 현재 서브볼륨으로 복사하는 것으로도 개별 파일 복원이 가능하다.

어떤 파일을 완전히 삭제하기 위해선 서브볼륨과 서브볼륨으로 생성한 모든 스냅샷에서 해당 파일을 지워야 한다. 다시 말하면, 모든 스냅샷에서 해당 파일을 지우지 않는 이상 여전히 디스크의 용량을 차지하고 있게 된다. 주기적으로 생성되는 스냅샷은 디스크 용량이 부족하거나 정리가 필요할 때 과감하게 지울 수 있어야 한다. 서브볼륨을 적절히 나누어 스냅샷 삭제 시 다른 서브볼륨에 영향이 없도록 한다.

## SSD - 207GB btrfs

```
/boot              1GB    ext4
swap               8GB    swap
/                 40GB    ext4
/mnt/ssd1-btrfs  207GB    btrfs
└-- @workspace
    ├-- user1
    └-- user2
```

ssd의 btrfs 공간은 개인 별 workspace 로 사용할 것이다.

`/home/user1/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user1`.

`/home/user2/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user2`.

@의 특별한 의미는 없다. 서브볼륨은 일반 디렉토리와 외관상 차이가 없기 때문에 단순히 서브볼륨이라는 것을 표현하기 위해 쓰인다.

## HDD - 1TB btrfs

```
/media          1024GB    ext4
/mnt/hdd1-btrfs 1024GB    btrfs
├-- @public
└-- @private
```

hdd는 사진이나 동영상, 문서 등 각종 데이터를 저장하는 용도로 사용할 것이다.

드라마, 영화 같은 콘텐츠는 중요하지 않기 때문에 ext4로 포맷된 /media에 저장할 것이고, 개인적인 사진, 문서 같이 중요한 것들은 btrfs 스냅샷을 이용한 데이터 백업 및 복원, 자체 압축을 위해 btrfs에 저장한다.

## 용도

정리하면 다음과 같다.

|서브볼륨|용도|마운트위치|스냅샷 생성 주기|
|:---:|:-----:|:---:|:---:|
|@workspace|개인 별 작업공간|/ws/\<user\>|30분|
|@public|공유 데이터|/public|-|
|@private|사적인 데이터|/home/jkpark/private|1일|

# btrfs 생성 과정

256GB SSD에 207GB만큼의 btrfs 을 생성하는 과정을 설명한다.  2TB HDD에 1TB만큼 btrfs 생성 또한 동일하게 진행하면 된다.


## 파티션 확인

`fdisk -l` 명령어로 현재 파티션을 볼 수 있다.


```
$ sudo fdisk -l
Disk /dev/sdb: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: ST1000DM003-1SB1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: dos
Disk identifier: 0x000102a4


Disk /dev/sda: 238.49 GiB, 256060514304 bytes, 500118192 sectors
Disk model: SAMSUNG MZNTY256
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x255bb064

Device     Boot    Start      End  Sectors  Size Id Type
/dev/sda1  *        2048  1953791  1951744  953M 83 Linux
/dev/sda2        1953792 17577983 15624192  7.5G 82 Linux swap / Solaris
/dev/sda3       17577984 95703039 78125056 37.3G 83 Linux
```

## 파티션 할당

`sda`에 192GB만큼의 btrfs를 할당한다.

```
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Command (m for help):
```

위와 같이 Command (m for help): 프롬프트가 떴으면 `F` 를 입력하여 빈 공간을 확인한다.

```
Command (m for help): F
Unpartitioned space /dev/sda: 192.86 GiB, 207060557824 bytes, 404415152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes

   Start       End   Sectors   Size
95703040 500118191 404415152 192.9G
```

> 원래 207GB여야 되는데 실제 시스템에는 192GB로 인식되어 192GB만큼이 생성되었다.

`N` 으로 새로운 파티션을 만든다.

```
Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): p

Selected partition 4
First sector (95703040-500118191, default 95703040):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (95703040-500118191, default 500118191):

Created a new partition 4 of type 'Linux' and of size 192.9 GiB.
```

파티션이 생성되었으면, `p`로 파티선 정보를 확인한다.

```
Command (m for help): p
Disk /dev/sda: 238.49 GiB, 256060514304 bytes, 500118192 sectors
Disk model: SAMSUNG MZNTY256
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x255bb064

Device     Boot    Start       End   Sectors   Size Id Type
/dev/sda1  *        2048   1953791   1951744   953M 83 Linux
/dev/sda2        1953792  17577983  15624192   7.5G 82 Linux swap / Solaris
/dev/sda3       17577984  95703039  78125056  37.3G 83 Linux
/dev/sda4       95703040 500118191 404415152 192.9G 83 Linux
```

`/dev/sda4`에 새로운 파티션이 생성된 것을 볼 수 있다.

> 경우에 따라 파티션 타입을 `LVM`으로 지정하고 `LVM` 안에 btrfs 볼륨을 만들기도 한다.
> 단순하게 설명하기는 어렵지만 raid 구성을 한다면 LVM으로 감싸는 쪽이 더 좋을 것이다. 나는 raid 구성을 안하기 떄문에 LVM으로 할 이유가 없고, raw partition으로 만드는 것이 LVM으로 감싸는 것보다 조금이라도 빠를 거라 믿는다.

`w` 명령어로 변경사항을 저장하고 나온다.

```
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

파티션을 만들었다면 파티션 테이블 변경사항을 커널에 알려야 한다.

```
$ sudo partprobe -s
/dev/sda: msdos partitions 1 2 3 4
/dev/sdb: msdos partitions
```

## make btrfs filesystem

`mkfs.btrfs` 명령어로 `/dev/sda4` 를 btrfs 파일시스템으로 만든다.

```
$ sudo mkfs.btrfs /dev/sda4
[sudo] password for jkpark:
btrfs-progs v5.4.1
See http://btrfs.wiki.kernel.org for more information.

Detected a SSD, turning off metadata duplication.  Mkfs with -m dup if you want to force metadata duplication.
Label:              (null)
UUID:               2767f408-217b-4d9b-8e44-611c47334ccc
Node size:          16384
Sector size:        4096
Filesystem size:    192.84GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         single            8.00MiB
  System:           single            4.00MiB
SSD detected:       yes
Incompat features:  extref, skinny-metadata
Checksum:           crc32c
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1   192.84GiB  /dev/sda4
```



생성이 끝났다.

### 마찬가지로 2TB HDD에 1TB만큼 btrfs 생성 또한 동일하게 진행하면 된다.

생성 결과

```
$ sudo mkfs.btrfs /dev/sdb1
btrfs-progs v5.4.1
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               972984bb-548f-4cc6-9c54-012159766391
Node size:          16384
Sector size:        4096
Filesystem size:    931.51GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP               1.00GiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Checksum:           crc32c
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1   931.51GiB  /dev/sdb1

```


# 마운트

`/dev/sda4`을 `/mnt/ssd1_btrfs`에 마운트한다.

```
$ sudo mkdir /mnt/ssd1_btrfs
$ sudo mount /dev/sda4 /mnt/ssd1_btrfs
```

`/dev/sdb1`을 `/mnt/hdd1_btrfs`에 마운트한다.

```
$ sudo mkdir hdd1_btrfs
$ sudo mount /dev/sdb1 /mnt/hdd1_btrfs
```


`df` 명령어로 마운트된 디스크 정보를 확인한다.

```
$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
/dev/sda4      btrfs     193G  3.4M  193G   1% /mnt/ssd1_btrfs
/dev/sdb1      btrfs     932G  3.8M  930G   1% /mnt/hdd1_btrfs
```

# 서브볼륨

서브볼륨은 블록 디바이스가 아니라 독립적인 파일트리 구조를 지닌 파일시스템이다. btrfs는 서브볼륨 단위의 스냅샷을 생성할 수 있는데 스냅샷 또한 이전 서브볼륨으로 부터 생성된 또 하나의 서브볼륨이다.

서브볼륨은 일반 디렉토리처럼 생겼지만 스냅샷을 생성하거나, 별도로 마운트를 할 수 있는 등 추가적인 동작을 할 수 있다. 방금 말한 것처럼 일반 디렉토리와 똑같이 생겼기 때문에 @을 붙여 서브볼륨이라는 것을 표현한다.

## 서브볼륨 생성

```
$ cd /mnt/ssd1_btrfs
$ sudo btrfs subvolume create @workspace
Create subvolume './@workspace'
```

```
$ cd /mnt/hdd1_btrfs/
$ sudo btrfs subvolume create @private
Create subvolume './@private'
$ sudo btrfs subvolume create @public
Create subvolume './@public'
```

## 서브볼륨 리스트

```
$ sudo btrfs subvolume list /mnt/ssd1_btrfs
ID 256 gen 7 top level 5 path @workspace
```

```
$ sudo btrfs subvolume list /mnt/hdd1_btrfs
ID 257 gen 7 top level 5 path @private
ID 258 gen 8 top level 5 path @public
```

top-level은 5부터 시작한다. 이 최상위 서브볼륨은 다른 서브볼륨에 의해 지워지거나 이동되지 않는다.

## Ownership

```
$ sudo chmod 700 @private
$ sudo chown jkpark: @private @public
```

- @private는 나만 사용할 수 있게 한다.
- @public은 나만 write 가능하고, 모두가 볼 수 있게 했다.
- @workspace는 하위에 개인 별 디렉토리 생성 예정이므로 건들지 않는다.


```
$ ls -l /mnt/hdd1_btrfs
drwx------ 1 jkpark jkpark  0 Dec 18 05:05 @private
drwxr-xr-x 1 jkpark jkpark  0 Dec 18 05:05 @public

$ ls -l /mnt/ssd1_btrfs
drwxr-xr-x 1 root root 0 Dec 18 05:04 @workspace
```

## 서브볼륨 마운트

원하는 위치에 마운트한다.

|서브볼륨|마운트 위치|
|:---:|:-----:|
|@workspace|/ws|
|@public|/public|
|@private|/home/jkpark/private|

```
$ sudo mkdir /ws
$ sudo mount -o subvol=@workspace /dev/sda4 /ws
```

```
$ mkdir /home/jkpark/private
$ mkdir /public
$ sudo mount -o subvol=@public /dev/sdb1 /public
$ sudo mount -o subvol=@private /dev/sdb1 /home/jkpark/private
```

> 언마운트는 `umount` 명령어로 한다.
> 
> ```
> $ sudo umount /public
> ```

### 마운트 자동화

btrfs 최상위 볼륨과 서브볼륨들이 부팅 시 자동으로 마운트되도록 설정한다.

먼저 btrfs 볼륨의 UUID를 확인한다.

```
$ ll /dev/disk/by-uuid/
lrwxrwxrwx 1 root root  10 Dec 18 04:27 2767f408-217b-4d9b-8e44-611c47334ccc -> ../../sda4
lrwxrwxrwx 1 root root  10 Dec 18 04:28 972984bb-548f-4cc6-9c54-012159766391 -> ../../sdb1
```

`/etc/fstab` 파일에 아래와 같이 마운트 설정을 한다.

```
# btrfs mount. 2020-12-17 jkpark
UUID=2767f408-217b-4d9b-8e44-611c47334ccc /mnt/ssd1_btrfs      btrfs   defaults,compress=lzo 0  2
UUID=2767f408-217b-4d9b-8e44-611c47334ccc /ws                  btrfs   defaults,compress=lzo,subvol=@workspace 0  0
UUID=972984bb-548f-4cc6-9c54-012159766391 /mnt/hdd1_btrfs      btrfs   noatime,nodiratime 0  2
UUID=972984bb-548f-4cc6-9c54-012159766391 /home/jkpark/private btrfs   noatime,nodiratime,subvol=@private 0  0
UUID=972984bb-548f-4cc6-9c54-012159766391 /public  btrfs   noatime,nodiratime,subvol=@public  0  0
```

마운트 옵션을 살펴보면,

- compress=lzo : lzo 로 압축한다. HDD에 압축옵션을 주면 성능이 떨어지므로 SSD만 주었다.
- noatime : access time을 기록하지 않는다.
- nodiratime : directory에 access time을 기록하지 않는다.

잘 모르겠다면, `defaults`만 주어도 된다.

맨 뒤 `0 2`는 각 각 dump와 pass 옵션이다.

- dump : Enable or disable backing up of the device/partition (the command dump). This field is usually set to 0, which disables it.
- pass : Controls the order in which fsck checks the device/partition for errors at boot time. The root device should be 1. Other partitions should be 2, or 0 to disable checking.

더 자세한 내용은 wifi 참고. https://help.ubuntu.com/community/Fstab

reboot 후 `findmnt` 명령어로 마운트가 잘 되는지 확인한다.

```
$ findmnt -nt btrfs
/ws                  /dev/sda4[/@workspace] btrfs  rw,relatime,compress=lzo,ssd,space_cache,subvolid=256,subvol=/@workspace
/mnt/ssd1_btrfs      /dev/sda4              btrfs  rw,relatime,compress=lzo,ssd,space_cache,subvolid=5,subvol=/
/public  /dev/sdb1[/@public]    btrfs  rw,noatime,nodiratime,space_cache,subvolid=258,subvol=/@public
/home/jkpark/private /dev/sdb1[/@private]   btrfs  rw,noatime,nodiratime,space_cache,subvolid=257,subvol=/@private
/mnt/hdd1_btrfs      /dev/sdb1              btrfs  rw,noatime,nodiratime,space_cache,subvolid=5,subvol=/
```

# 스냅샷

주기적으로 스냅샷을 만들건데, 위치는 스냅샷을 만들 서브볼륨 밖으로 지정한다.

스냅샷과 서브볼륨의 구조는 다음과 같다.

```
/mnt/ssd1-btrfs
├-- @workspace
└-- snapshots
    └-- workspace
        ├-- @workspace_2020.12.18  (snapshot of @workspace)
        └-- @workspace_2020.12.19  (snapshot of @workspace)

/mnt/hdd1-btrfs
├-- @public
├-- @private
└-- snapshots
    └-- public
    │   ├-- @public_2020.12.18  (snapshot of @public)
    │   └-- @public_2020.12.19  (snapshot of @public)
    └-- private
        ├-- @private_2020.12.18  (snapshot of @private)
        └-- @private_2020.12.19  (snapshot of @private)
```

스냅샷 디렉토리를 만든다.

```
$ sudo mkdir -p /mnt/ssd1_btrfs/snapshots/workspace
$ sudo mkdir -p /mnt/hdd1_btrfs/snapshots/public /mnt/hdd1_btrfs/snapshots/private
```

## 스냅샷 생성

```
$ sudo btrfs subvolume snapshot /mnt/ssd1_btrfs/@workspace /mnt/ssd1_btrfs/snapshots/workspace/@workspace_`date +%Y.%m.%d_%H.%M.%S`
Create a snapshot of '/mnt/ssd1_btrfs/@workspace' in '/mnt/ssd1_btrfs/snapshots/workspace/@workspace_2020.12.19_00.08.07'
```

`date +%Y.%m.%d_%H.%M.%S` 는 연.월.일_시.분.초 형식으로 저장하라는 명령이다. 반드시 이렇게 작성해야 samba를 통해 윈도우OS에서 스냅샷을 인식하고 복원할 수 있다.

스냅샷이 생성되었는지 확인한다.

```
$ sudo btrfs subvolume list /mnt/ssd1_btrfs
ID 256 gen 12 top level 5 path @workspace
ID 257 gen 12 top level 5 path snapshots/workspace/@workspace_2020.12.19_00.08.07
```

스냅샷 `snapshots/workspace/@workspace_2020.12.19_00.08.07` 이 생성된 것을 볼 수 있다.

## 스냅샷으로 복원

1. 현재 서브볼륨을 스냅샷으로 복원하기에 앞서 현재 디렉토리를 백업한다.

```
$ sudo mv /mnt/ssd1_btrfs/@workspace /mnt/ssd1_btrfs/snapshots/workspace/@workspace_latest
```

2. 복원할 스냅샷을 서브볼륨 이름으로 이동시킨다.

```
$ sudo mv /mnt/ssd1_btrfs/snapshots/workspace/@workspace_2020.12.19_00.08.07 /mnt/ssd1_btrfs/@workspace
```

리마운트한다.

```
$ sudo umount /ws
$ sudo mount -o defaults,compress=lzo,subvol=@workspace /dev/sda4 /ws
```

### 개별 파일 복원

특정 파일만 복원을 원한다면, 원하는 시점의 스냅샷에서 파일을 복사해오면 된다.

`@workspace`의 스냅샷 `@workspace_2020.12.19_00.08.0`에서 `helloworld` 파일을 복원할 경우,

```
$ cp /mnt/ssd1_btrfs/snapshots/workspace/@workspace_2020.12.19_00.08.07/helloworld /ws/helloworld
```

### 윈도우에서 파일 복원



## 스냅샷 삭제

```
$ sudo btrfs subvolume delete /mnt/ssd1_btrfs/snapshots/workspace/@workspace_2020.12.19_00.08.07
Delete subvolume (no-commit): '/mnt/ssd1_btrfs/snapshots/workspace/@workspace_2020.12.19_00.08.07'
```


## 자동화