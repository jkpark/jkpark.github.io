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

128GB SSD에는 79GB만큼 btrfs를 할당하고, 2TB HDD에는 1TB만큼 btrfs를 할당할 것이다.

btrfs는 자신의 볼륨을 여러개의 서브볼륨 형태로 나눌 수 있기 때문에 디스크 전체를 btrfs 파티션으로 생성하고 서브볼륨 단위로 나눈다. 파티션의 경우 크기을 정해주어야 하는 것과 달리 btrfs의 서브볼륨은 용량에 제한이 없다. 또한, 각각의 서브볼륨을 스냅샷으로 저장해두면 언제든지 저장시점으로 되돌릴 수 있다. 스냅샷 이름의 형식을 윈도우 파일 버전 관리 이름 형식으로 마춰준다면 윈도우 파일 탐색기를 통해서도 복원이 가능하다.

복원시점인 스냅샷은 서브볼륨 단위로 생성하고 삭제할 수 있다. 스냅샷에는 해당 시점에 있는 모든 파일들이 담겨있어서 스냅샷을 생성해 두면 파일을 삭제하여도 언제든지 복원이 가능하다. 해당 스냅샷 안에 있는 파일을 현재 서브볼륨으로 복사하는 것으로도 개별 파일 복원이 가능하다.

어떤 파일을 완전히 삭제하기 위해선 서브볼륨과 서브볼륨으로 생성한 모든 스냅샷에서 해당 파일을 지워야 한다. 다시 말하면, 모든 스냅샷에서 해당 파일을 지우지 않는 이상 여전히 디스크의 용량을 차지하고 있게 된다. 주기적으로 생성되는 스냅샷은 디스크 용량이 부족하거나 정리가 필요할 때 과감하게 지울 수 있어야 한다. 서브볼륨을 적절히 나누어 스냅샷 삭제 시 다른 서브볼륨에 영향이 없도록 한다.

## SSD - 79GB btrfs

```
├── /boot              1GB    ext4
├── swap               8GB    swap
├── /                 40GB    ext4
└── /mnt/ssd1-btrfs   79GB    btrfs
       └── @workspace
              ├── user1
              └── user2
```

ssd의 btrfs 공간은 개인 별 workspace 로 사용할 것이다.

`/home/user1/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user1`.

`/home/user2/workspace` will be linked to `/mnt/ssd1-btrfs/@workspace/user2`.

## HDD - 1TB btrfs

```
├── /media          1024GB    ext4
└── /mnt/hdd1-btrfs 1024GB    btrfs
       ├── @public
       └── @private
```

hdd는 사진이나 동영상, 문서 등 각종 데이터를 저장하는 용도로 사용할 것이다.

드라마, 영화 같은 콘텐츠는 중요하지 않기 때문에 ext4로 포맷된 /media에 저장할 것이고, 개인적인 사진, 문서 같이 중요한 것들은 btrfs 스냅샷을 이용한 데이터 백업 및 복원, 자체 압축을 위해 btrfs에 저장한다.

## 용도

정리하면 다음과 같다.

|서브볼륨|용도|스냅샷 생성 주기|
|:---:|:-----:|:---:|
|workspace|개인 별 작업공간|30분|
|public|공유 데이터|-|
|private|사적인 데이터|1일|

# 포맷 과정