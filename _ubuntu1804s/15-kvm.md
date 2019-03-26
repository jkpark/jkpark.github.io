---
layout: post
title: 15장. 가상머신 - QEMU-KVM
description: 
category: ubuntu1804
english: false
---

# Overview

Ubuntu 18.04에서 VM을 띄워 윈도우10을 설치할 것이다. 
Display는 VNC로 설정한다.

# check if the system supports hardware virtualization

```
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

# install packages

```
$ sudo apt-get install qemu-kvm libvirt-bin virtinst
```

# chekc VMs

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


# 방화벽 설정

VNC를 통해 Display를 접근할 것이기 때문에 VNC 방화벽 포트를 열어준다.

```
$ sudo iptables -A INPUT -s 172.17.0.0/16 -p tcp -m tcp --dport 5900 -j ACCEPT
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

`172.17.0.0/16`은 docker의 NAT주소이다. guacamole의 네트워크가 이 서브넷 안에 존재하기 때문에 해당 주소에서의 접속을 허용해준다.


# Create VM

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
--disk path=/var/lib/libvirt/images/vm-win10.qcow2,format=qcow2,size=40 \
--graphics vnc,port=5900,listen=0.0.0.0
```

- `hvm` : Request the full hardware virtualization.
- `graphics vnc,port=5900,listen=0.0.0.0` : Allows VNC access to the VM from a remote client.
- `cdrom /home/jkpark/workspace/Win10.iso` : Install ISO image
- `disk path=/var/lib/libvirt/images/vm-win10.qcow2,format=qcow2,size=40` : 파일 경로 및 크기

명령어를 실행하면 가상 머신이 실행된다. guacamole을 설치하였으니 웹 브라우저에서 vnc로 접속해본다.

![access via VNC](/images/ubuntu1804/guacamole02.png)

윈도우10 가상머신을 설치 하였다면 `virsh list` 명령어를 통해 VM의 상태를 볼 수 있다.

```
$ virsh list
 Id    Name                           State
----------------------------------------------------
 7     win10                          running
```

VM마다 ID가 있다. 이 ID로도 명령어를 내릴 수 있다.

```
$ virsh shutdown 10
Domain 7 is being shutdown
$ virsh list
 Id    Name                           State
----------------------------------------------------
 7     win10                          in shutdown
```

# IP 고정

libvirt는 기본 네트워크인 `default`는 `192.168.122.xxx` 대역을 쓰고 있다.

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
$ virsh dumpxml win10 | grep 'mac address'
      <mac address='52:54:00:c2:aa:53'/>

$ virsh net-update default add ip-dhcp-host "<host mac='52:54:00:c2:aa:53' name='win10' ip='192.168.122.2' />" --live --config
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

해제

```
$ virsh autostart win10 --disable
```

# Remove VM

```
$ virsh shutdown win10
$ virsh destroy win10
$ virsh undefine win10
$ sudo rm /var/lib/libvirt/images/vm-win10.qcow2
$ sudo virsh net-update default delete ip-dhcp-host "<host mac='52:54:00:c2:aa:53' name='win10' ip='192.168.122.2' />" --live --config
```