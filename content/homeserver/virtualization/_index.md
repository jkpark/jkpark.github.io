---
title: Virtualization
description:
date: 2020-12-21T11:41:54+09:00
draft: false
weight: 0
image: "" # relative path of /static/images folder
collapse: hide # show | hide | always
type: docs
---

# qemu-kvm

보통 가상머신은 VMware나 Vitual-Box를 떠올리지만, 리눅스의 대표적인 가상 머신은 `QEMU-KVM` 이다. `QEMU`와 `KVM`은 각각 Hypevisor 중 하나인데, 둘의 장점을 합쳐 사용하는 것이 `QEMU-KVM`이다.

이 둘의 개념을 설명하려면 하이퍼바이저는 무엇인지, 전가상화와 반가상화는 무엇인지, 에뮬레이션과 시뮬레이션의 차이는 무엇인지 등 알아야 할 지식이 끝이 없기 때문에 자세한 설명은 생략하고 `QEMU`와 `KVM`을 같이 쓰는지 이유만 간단히 설명한다.

먼저 에뮬레이션이란, 가상화에 필요한 하드웨어를 소프트웨어적으로 구현하여 가상머신에게 제공하는 것이다. `QEMU`는 에뮬레이터이다보니 동작에 필요한 하드웨어를 독립적으로 생성하기 때문에 성능이 떨어진다.

반면 시뮬레이터인 `KVM`은 호스트 머신에서 제공하는 하드웨어를 그대로 쓰기 때문에 에뮬레이터에 비해 상대적으로 성능이 좋다. 하지만 가상화에 필요한 하드웨어가 없다면 시뮬레이션에 제약이 생긴다.

이러한 시뮬레이터와 에뮬레이터의 이점을 합쳐 `QEMU-KVM`는 호스트와 게스트가 동일한 아키텍처를 사용하면 `KVM`을 통해 하드웨어를 가속화 하고 `KVM`에서 제공하지 못하는 하드웨어들은 `QEMU`에 의해 에뮬레이션하도록 한다.

## KVM이 가능한지 확인

```
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

## 설치

```
$ sudo apt install qemu-kvm
```

# libvirt

`libvirt`는 가상머신 스택의 핵심이다. VM을 만들고, 지우고 관련 네트워크를 관리하는 등 다양한 서비스를 `libvirt-daemon`이 수행하게 된다.

`virtinst`는 가상머신 설치할 때 쓰인다.

```
$ sudo apt install libvirt-daemon-system virtinst
```

# check VMs

`virsh`는 `libvirt`의 UI이다.

```
$ sudo virsh list --all
 Id    Name                           State
----------------------------------------------------
```

아직 VM을 생성한 적이 없으므로 빈 리스트가 출력된다.

```
$ sudo virsh net-list --all
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     yes           yes
```

default가 안뜬다면 virbr0을 재시작해본다.

```
$ sudo ifconfig virbr0 down
$ sudo ifconfig virbr0 up
```

# Create VM

Windows 10용 VM 생성 방법이다.

```
$ sudo virt-install \
--virt-type kvm \
--name win10 \
--vcpus 2 \
--cpu host-passthrough,cache.mode=passthrough \
--hvm \
--memory 2048 \
--os-variant win10 \
--cdrom /home/jkpark/workspace/Win10.iso \
--disk path=/home/jkpark/public/vm-win10.qcow2,format=qcow2,size=40 \
--graphics vnc,port=5900,listen=0.0.0.0
```

- `--cpu host-passthrough,cache.mode=passthrough` : 가상머신이 실제 CPU정보를 그대로 사용
- `--os-variant` : win10, ubuntu20.04 등 설치하는 OS 종류
- `--graphics vnc,port=5900,listen=0.0.0.0` : VNC를 통해 Display를 접근할 것이기 때문에 VNC 방화벽 포트 5900번을 열어준다.
  VNC는 보안에 취약하므로, `-s x.x.x.x`와 같이 접속할 source를 지정하여 보안을 강화한다.
  ```
  $ sudo iptables -A INPUT -s x.x.x.x -p tcp -m tcp --dport 5900 -j ACCEPT
  $ sudo netfilter-persistent save
  ```

vm이 할당이 되면 `virsh list` 명령어를 통해 VM의 상태를 볼 수 있다.

```
$ virsh list
 Id    Name                           State
----------------------------------------------------
 7     win10                          running
```

VM마다 ID가 있다. 이 ID로도 명령어를 내릴 수 있다.

```
$ virsh shutdown 7
Domain 7 is being shutdown
$ virsh list
 Id    Name                           State
----------------------------------------------------
 7     win10                          in shutdown
```

# 접속

VNC viewer for Chrome 을 통해 접속한다.

https://chrome.google.com/webstore/detail/vnc%C2%AE-viewer-for-google-ch/iabmpiboiopbgfabjmgeedhcmjenhbla

# 주요 virsh 명령어

```
virsh list --all

virsh start <domain>

virsh shutdown  <domain>

virsh undefine  <domain>
디스크는 삭제되지 않으므로 직접 삭제해야 한다.

virsh autostart  <domain>
호스트 부팅 시 자동 시작
```

# 고정 IP

libvirt는 기본 네트워크 default는 `192.168.122.xxx` 대역을 쓰고 있다.

```
$ virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>84fffe88-e89a-4da3-9e47-c705bb76d404</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:90:85:c3'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
    </dhcp>
  </ip>
</network>
```

이 기본 네트워크가 VM에게 고정 IP를 내려주도록 설정한다.

```
$ virsh dumpxml <domain> | grep 'mac address'
      <mac address='52:54:00:c2:aa:53'/>

$ virsh net-update default add ip-dhcp-host "<host mac='52:54:00:c2:aa:53' name='<domain>' ip='192.168.122.2' />" --live --config
Updated network default persistent config and live state
```

확인

```
$ virsh net-dumpxml default
<network>
  <name>default</name>
  <uuid>e7dd4a92-a610-4e34-979f-acc36ac3a9a1</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr0' stp='on' delay='0'/>
  <mac address='52:54:00:bd:ba:d4'/>
  <ip address='192.168.122.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.122.2' end='192.168.122.254'/>
      <host mac='52:54:00:c2:aa:53' name='win10' ip='192.168.122.2'/>
    </dhcp>
  </ip>
</network>
```

설정 후 VM을 재시작한다.

```
$ virsh shutdown win10
$ virsh start win10
```

우분투 서버 부팅 시 자동으로 시작되도록 설정하려면 아래 명령어를 입력한다.

```
$ virsh autostart win10
```

[참고] 해제

```
$ virsh autostart win10 --disable
```

# Remove VM

win10 예:

```
$ virsh shutdown win10
$ virsh destroy win10
$ virsh undefine win10
$ sudo rm /var/lib/libvirt/images/vm-win10.qcow2
$ sudo virsh net-update default delete ip-dhcp-host "<host mac='52:54:00:c2:aa:53' name='win10' ip='192.168.122.2' />" --live --config
```
