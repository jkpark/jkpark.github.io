---
layout: post
title: 12장. 가상머신 - QEMU-KVM
description: 
category: ubuntu1804
english: false
---

# Overview

Ubuntu 18.04에서 VM을 띄워 rhel 7.6을 설치할 것이다. 
Display는 VNC로 설정할 것이다. 

# check if the system supports hardware virtualization

```
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```

# install packages

```
sudo apt-get install qemu-kvm libvirt-bin virtinst
```

# chekc VMs

```
$ sudo virsh list --all
 Id    Name                           State
----------------------------------------------------

```

아직 VM을 생성한 적이 없으므로 빈 리스트가 출력된다.


# 방화벽 설정

VNC를 통해 Display를 접근할 것이기 때문에 VNC 방화벽 포트를 열어준다.

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 5900 -j ACCEPT
$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
```

# Create VM

```
$ sudo virt-install \
--virt-type kvm \
--name rhel7.6 \
--vcpus 4 \
--cpu host-passthrough,cache.mode=passthrough \
--hvm \
--memory 4048 \
--os-variant rhel7 \
--cdrom /home/jkpark/Downloads/rhel-server-7.6-x86_64-dvd.iso \
--disk path=/var/lib/libvirt/images/vm1-rhel-server-7.6.qcow2,format=qcow2,size=40 \
--graphics vnc,port=5900,listen=0.0.0.0
```

- `hvm` : Request the full hardware virtualization.
- `graphics vnc,port=5900,listen=0.0.0.0` : Allows VNC access to the VM from a remote client.
- `cdrom /home/jkpark/Downloads/rhel-server-7.6-x86_64-boot.iso` : Install ISO image
- `disk path=/var/lib/libvirt/images/vm1-rhel-server-7.6.qcow2,format=qcow2,size=40` : 파일 경로 및 크기

명령어를 실행하면 가상 머신이 실행된다. VNC를 통해 접속하면 아래와 같이 rhel7.6 설치화면을 볼 수 있다.

![](/images/posts/kvm-rhel7.6-vnc01.png)

![](/images/posts/kvm-rhel7.6-vnc02.png)

![](/images/posts/kvm-rhel7.6-vnc03.png)

![](/images/posts/kvm-rhel7.6-vnc03.png)

설치가 끝났다. `virsh list` 명령어를 통해 VM의 상태를 볼 수 있다.

```
$ virsh list
 Id    Name                           State
----------------------------------------------------
 10    rhel7.6                        running
```

VM마다 ID가 있다. 이 ID로도 명령어를 내릴 수 있다.

```
$ virsh shutdown 10
Domain 10 is being shutdown
$ virsh list
 Id    Name                           State
----------------------------------------------------
 10    rhel7.6                        in shutdown
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
$ virsh dumpxml rhel7.6 | grep 'mac address'
      <mac address='52:54:00:a3:f8:c6'/>

$ virsh net-update default add ip-dhcp-host "<host mac='52:54:00:a3:f8:c6' name='rhel7.6' ip='192.168.122.2' />" --live --config
Updated network default persistent config and live state

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
      <host mac='52:54:00:a3:f8:c6' name='rhel7.6' ip='192.168.122.2'/>
    </dhcp>
  </ip>
</network>
```

설정 후 VM을 재시작한다.

```
$ virsh shutdown rhel7.6
$ virsh start rhel7.6
```

우분투 서버 부팅 시 자동으로 시작되도록 설정한다.

```
$ virsh autostart rhel7.6
```

# Remove VM

```
$ virsh shutdown rhel7.6
$ virsh destroy rhel7.6
$ virsh undefine rhel7.6
$ sudo rm /var/lib/libvirt/images/vm1-rhel-server-7.6.qcow2
$ sudo virsh net-update default delete ip-dhcp-host "<host mac='52:54:00:a3:f8:c6' name='rhel7.6' ip='192.168.122.2' />" --live --config
```